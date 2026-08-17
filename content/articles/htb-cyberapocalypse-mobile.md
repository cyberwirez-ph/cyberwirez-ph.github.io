---
title: "HTB Cyber Apocalypse - Mobile Track Writeup"
date: 2026-08-10
description: "A technical writeup for the Mobile track of HTB Cyber Apocalypse 2026 -- reverse-engineering the cryptographic checks hidden inside three Godot-engine Android games."
author: "drewbyte"
draft: false
---

*Originally published on [iamdrewbyte.github.io](https://iamdrewbyte.github.io/posts/htbcyberapocalypse-mobile/)*

**Challenges:** Overstrike (easy) · Proofmark (medium) · SaltCrown (hard)

![Mobile category](/images/articles/htb-cyberapocalypse-mobile/00_scoreboard.png)

All three are **Godot-engine Android games**. None of them are really “games” - each one hides a cryptographic check that decides whether you win, and the intended solve is to find that check, understand its math, and then either invert it or compute the answer directly.

The same three-step shape applies to all three:

  1. Find where the win condition is actually computed.
  2. Read the exact math.
  3. Invert it, or brute-force the small unknown.

* * *

## Overstrike

### Summary

You play a character carrying a “Mark.” The world has a “True Seal.” The game constantly takes your Mark, runs it through a scrambler, and checks whether the scrambled result equals the True Seal.

![Registry HUD](/images/articles/htb-cyberapocalypse-mobile/02_ov_registry_hud.png)

The trick: the scrambler the game uses is a **well-known public algorithm that can be run backwards**. So instead of hunting for the right Mark, you take the answer the game wants, run the scrambler in reverse, and it hands you the exact Mark you need. Then a second function uses that Mark as a decryption key to reveal the flag - and since we now know the Mark, we can just run that decryption ourselves on our own machine. The game never needs to be played.

* * *

## Proof of Concept (Technical)

### Step 1 - Identify the runtime

`apktool d Overstrike.apk` shows this is a **Godot Mono (C#)** build. The `assets/scripts/*.cs` files on disk are empty stubs, but the compiled managed assembly is shipped intact:

```text
./Overstrike/assets/.godot/mono/publish/arm64/Overstrike.dll
```
 

So no native reversing needed - this opens cleanly in **ILSpy**.

![ILSpy class tree](/images/articles/htb-cyberapocalypse-mobile/03_ov_ilspy_tree.png)

The class names match the empty stubs exactly: `Archive`, `BridgeBuilder`, `GameState`, `MarkPickup`, etc.

### Step 2 - Read `GameState`

```csharp
public ulong CarriedMark;
public ulong WorldSeal;
public const ulong TrueSeal = 15682021040575554950uL;

public bool WorldIsAligned => WorldSeal == 15682021040575554950uL;

public override void _Process(double delta)
{
    WorldSeal = Mix(CarriedMark);          // recomputed every frame
}

public static ulong Mix(ulong x)
{
    long num  = (long)x + -7046029254386353131L;
    long num2 = (num  ^ (num  >>> 30)) * -4658895280553007687L;
    long num3 = (num2 ^ (num2 >>> 27)) * -7723592293110705685L;
    return (ulong)(num3 ^ (num3 >>> 31));
}
```
 

Converting those signed decimals to unsigned hex:

Signed decimal | Unsigned hex  
---|---  
`-7046029254386353131` | `0x9E3779B97F4A7C15`  
`-4658895280553007687` | `0xBF58476D1CE4E5B9`  
`-7723592293110705685` | `0x94D049BB133111EB`  
  
Those are the **splitmix64** constants verbatim. splitmix64 is a bijection on 64-bit integers, so it has an exact inverse - no search required.

### Step 3 - Confirm the intended path is impossible

`MarkPickup.cs` is how the Mark is supposed to grow:

```csharp
[Export] public ulong Worth = 1uL;

private void OnBodyEntered(Node3D body)
{
    if (body is Player && GameState.Instance != null)
    {
        GameState.Instance.CarriedMark += Worth;   // small increments only
        QueueFree();
    }
}
```
 

Small additive pickups can never reach a ~1.5×10¹⁹ target. The value must be reached by computation, not by play.

### Step 4 - Invert splitmix64

Each stage is individually invertible: multiplications are odd (so invertible mod 2⁶⁴ via modular inverse), and `x ^= x >> k` unwinds by iterating.

```python
MASK = (1 << 64) - 1

def inv_xorshift(z, shift):
    x = z
    for _ in range(64 // shift + 1):
        x = z ^ (x >> shift)
    return x & MASK

C1 = 0x9e3779b97f4a7c15
C2 = 0xbf58476d1ce4e5b9
C3 = 0x94d049bb133111eb
MOD = 1 << 64
inv_C2, inv_C3 = pow(C2, -1, MOD), pow(C3, -1, MOD)

TrueSeal = 15682021040575554950

def invert_splitmix64(out):
    z = out
    z = inv_xorshift(z, 31)
    z = (z * inv_C3) & MASK
    z = inv_xorshift(z, 27)
    z = (z * inv_C2) & MASK
    z = inv_xorshift(z, 30)
    return (z - C1) & MASK

print(invert_splitmix64(TrueSeal))
```
 
```text
Recovered CarriedMark: 15549431037298259574
Hex: 0xd7caad24dd98b676
Forward check: 15682021040575554950 == TrueSeal? True
```
 

### Step 5 - Decrypt the flag offline

`UnsealRegistry()` is a **pure function** of `CarriedMark` and a hardcoded 56-byte blob - a SHA-256 counter-mode keystream XORed against `SealedRecord`. Nothing runtime-dependent, so it can be reimplemented directly:

```python
import hashlib, struct

CarriedMark = 15549431037298259574
SealedRecord = bytes([13,86,51,68,18,110,68,15,54,61,236,94,135,202,213,182,4,1,182,181,
    150,228,184,126,121,224,236,220,7,82,153,251,179,104,0,87,32,34,3,60,
    166,96,124,50,253,31,124,179,220,157,120,115,19,47,96,11])

def unseal_registry(mark):
    key = hashlib.sha256(struct.pack('<Q', mark)).digest()
    out = bytearray(len(SealedRecord))
    n = ctr = 0
    while n < len(SealedRecord):
        block = hashlib.sha256(key + struct.pack('<i', ctr)).digest()
        j = 0
        while j < len(block) and n < len(SealedRecord):
            out[n] = SealedRecord[n] ^ block[j]
            j += 1; n += 1
        ctr += 1
    return ''.join(chr(b) if 32 <= b < 127 else '▯' for b in out)

print(unseal_registry(CarriedMark))
```
 

### Flag

```text
┌──(kali㉿DESKTOP-B2E2TUG)-[/mnt/c/Users/adm_consultant/Downloads/mobile_overstrike]
└─$ cat > solve_overstrike.py << 'EOF'
import hashlib, struct
MASK = (1 << 64) - 1

def inv_xorshift(z, shift):
    x = z
    for _ in range(64 // shift + 1):
        x = z ^ (x >> shift)
    return x & MASK

C1, C2, C3 = 0x9e3779b97f4a7c15, 0xbf58476d1ce4e5b9, 0x94d049bb133111eb
MOD = 1 << 64
TrueSeal = 15682021040575554950

z = inv_xorshift(TrueSeal, 31)
z = (z * pow(C3, -1, MOD)) & MASK
z = inv_xorshift(z, 27)
z = (z * pow(C2, -1, MOD)) & MASK
z = inv_xorshift(z, 30)
mark = (z - C1) & MASK
print("CarriedMark:", mark, hex(mark))

SealedRecord = bytes([13,86,51,68,18,110,68,15,54,61,236,94,135,202,213,182,4,1,182,181,
    150,228,184,126,121,224,236,220,7,82,153,251,179,104,0,87,32,34,3,60,
    166,96,124,50,253,31,124,179,220,157,120,115,19,47,96,11])

key = hashlib.sha256(struct.pack('<Q', mark)).digest()
out = bytearray(len(SealedRecord)); n = ctr = 0
while n < len(SealedRecord):
    blk = hashlib.sha256(key + struct.pack('<i', ctr)).digest()
    j = 0
    while j < len(blk) and n < len(SealedRecord):
        out[n] = SealedRecord[n] ^ blk[j]; j += 1; n += 1
    ctr += 1
print(''.join(chr(b) if 32 <= b < 127 else '?' for b in out))
EOF
python3 solve_overstrike.py
CarriedMark: 15549431037298259574 0xd7caad24dd98b676
HTB{0v3rstr1k3_r3cut_th3_w0rld_s34l_by_f0rg1ng_th3_mark}
```
 

The flag text itself confirms the intended solution - _“re-cut the world seal by forging the mark.”_ The emulator was never needed.

* * *

## Proofmark

### Summary

You bring a self-made ring to an inspection anvil. The anvil is supposed to tell a genuine heirloom from a counterfeit. Your job is to make a fake it cannot distinguish.

![Proofmark](/images/articles/htb-cyberapocalypse-mobile/10_proofmark_game.png)

This one hides its logic differently: the game scripts are readable, but they hand the actual decision off to a **compiled machine-code library**. Inside that library are two separate gates:

  1. Your ring’s _measurements_ must exactly match one hardcoded combination burned into the binary.
  2. Your ring’s _stamp_ must independently satisfy a second, much heavier check.

The measurements can simply be read out of the binary as raw bytes. The stamp is harder - but we never actually need it. The library’s decryption routine is a tiny, fast function of a single 32-bit number, and we already know the answer starts with `HTB{`. So we try all four billion possibilities until one produces that prefix. That takes under a minute on a laptop.

* * *

## Proof of Concept (Technical)

### Step 1 - Locate the logic

`apktool` shows this is **plain GDScript** (`.gdc` bytecode, no C# assemblies). Decompiling the pack with **GDRE Tools** recovers the scripts. `ForgeClient.gd` is the giveaway:

```gdscript
enum Verdict { REJECT_STATE, REJECT_TOKEN, ACCEPTED }

func reseal(state: PackedByteArray) -> int:
    return _native.reseal(state.decode_s32(0), state.decode_s32(4),
                          state.decode_s32(8), state.decode_s32(12)) & 4294967295

func strike(state: PackedByteArray, hallmark: int) -> Array:
    var v: int = _native.submit(state.decode_s32(0), state.decode_s32(4),
                                state.decode_s32(8), state.decode_s32(12), hallmark)
    var mark: String = _native.certificate() if v == Verdict.ACCEPTED else ""
    return [v, mark]
```
 

The real work lives in `lib/x86_64/libproofmark.x86_64.so`, a GDExtension declared by `proofmark.gdextension` with entry symbol `proofmark_library_init`.

And `GameState.gd` defines the state layout - four little-endian int32s:

```gdscript
var wards: PackedInt32Array = PackedInt32Array([0, 0, 0])
var bite: int = 0

func file_ward(index: int, delta: int) -> void:
    wards[index] = clampi(wards[index] + delta, 0, 24)
    bite = wards[0] + wards[1] * 2 + wards[2] * 3
    _resync_hallmark()

func snapshot() -> PackedByteArray:
    var buf := PackedByteArray(); buf.resize(16)
    buf.encode_s32(0,  wards[0]); buf.encode_s32(4,  wards[1])
    buf.encode_s32(8,  wards[2]); buf.encode_s32(12, bite)
    return buf
```
 

### Step 2 - Find the bound methods in Ghidra

Loading the `.so` and tracing `proofmark_library_init` → the initialize callback lands on the ClassDB registration block:

![Method registration](/images/articles/htb-cyberapocalypse-mobile/11_pm_method_reg.png)

`reseal`, `submit` and `certificate` are registered here, each with a call-wrapper and a ptrcall-wrapper function pointer.

### Step 3 - Follow `reseal` to its real implementation

The ptrcall wrapper for `reseal` (`FUN_00102bb0`) just re-packs the four int32 arguments back into a 16-byte buffer - the identical layout `snapshot()` produces - and calls into the real routine:

![reseal call](/images/articles/htb-cyberapocalypse-mobile/12_pm_reseal_call.png)

`FUN_00101c10(&local_28, 0x10)` is the hash core. Internally it’s a **custom bytecode-VM interpreter** running 100,000 iterations - deliberately painful to read statically.

### Step 4 - Read the acceptance condition

Skipping the VM body and reading only its tail reveals the whole game:

![Genuine vs forged branch](/images/articles/htb-cyberapocalypse-mobile/13_pm_genuine_branch.png)

```c
if (param_2 == 0x10) {
    if ( (*param_1 < 0x19) &&                       // ward0 in [0,24]
         (param_1[1] within [0,24]) &&               // ward1
         (param_1[2] within [0,24]) &&               // ward2
         (param_1[3] == param_1[2]*3 + *param_1 + param_1[1]*2) ) {   // bite formula
        return uVar10;                               // "genuine" path
    }
}
// otherwise: a completely different, much simpler finalizer
uVar7 = (((local_78[0] ^ 0xa5a5a5a5) >> 0x10) ^ local_78[0] ^ 0xa5a5a5a5) * -0x7a1435cb;
uVar7 = ((uVar7 >> 0xd) ^ uVar7) * -0x3d4d51cb;
return (uVar7 >> 0x10) ^ uVar7 ^ 0xbadf00d;
```
 

That last line is `bite == w0 + 2·w1 + 3·w2` \- **exactly** the formula in `GameState.gd`. The binary independently re-derives what a legitimate ring should look like. Note the `0xbadf00d` on the counterfeit path.

### Step 5 - Extract the hardcoded target state

`submit`’s real implementation (`FUN_001020d0`) opens with a 128-bit `MOVDQU`/`PXOR`/`PTEST` compare against `_DAT_00100530` \- meaning the state buffer must match one specific hardcoded 16 bytes. Reading them straight out of the data section:

![Target state bytes](/images/articles/htb-cyberapocalypse-mobile/15_pm_target_state.png)

```text
00100530:  53 00 00 00   43 00 00 00   37 00 00 00   ce 01 00 00
           wards[0]=83   wards[1]=67   wards[2]=55   bite=462
```
 

Sanity check: `83 + 2·67 + 3·55 = 83 + 134 + 165 = 382`… which is **not** 462\. The hardcoded state is deliberately _not_ internally consistent - it is the forgery. _(The first three bytes also spell`S C 7` in ASCII - an author nod.)_

### Step 6 - Read the certificate decryption

![submit core](/images/articles/htb-cyberapocalypse-mobile/14_pm_submit_core.png)

```c
iVar1 = 1200000;
do {                                     // 1.2M rounds of MurmurHash3-style mixing
    uVar4 = (param_3 + 0xc2b2ae35 >> 0x10 ^ param_3 + 0xc2b2ae35) * -0x7a143595;
    ...
} while (iVar1 != 0);
...
do {                                     // keystream → XOR against embedded table
    *(byte *)((long)local_38 + lVar2) = (byte)(uVar5 >> 0x18) ^ (&DAT_001006a0)[lVar2];
    lVar2 = lVar2 + 1;
} while (lVar2 != 0x1c);                 // 28 bytes
...
if (local_38[0] == 0x7b425448) {         // little-endian ASCII "HTB{"
    memcpy(param_4, local_38, ...);      // emit certificate
    uVar3 = 2;                           // Verdict.ACCEPTED
}
```
 

`0x85ebca6b` / `0xc2b2ae35` are the MurmurHash3 32-bit finalizer constants (shown negated by Ghidra).

### Step 7 - Confirm the state is correct, live

Rather than trust static reading, hook the real library in the running process with **Frida** and call the two internals directly. Addresses were located by **byte-pattern scan** (Ghidra’s displayed addresses carry a synthetic image base, so raw arithmetic offsets do not map to runtime):

```javascript
var mod        = Process.findModuleByName("libproofmark.x86_64.so");
// offsets confirmed empirically via Memory.scanSync for the function prologues
var resealAddr = mod.base.add(0x4c10);   // 53 48 83 EC 50        (push rbx; sub rsp,0x50)
var submitAddr = mod.base.add(0x50d0);   // 31 C0 83 FE 10 75 13  (xor eax,eax; cmp esi,0x10)

var reseal = new NativeFunction(resealAddr, 'uint32', ['pointer','int32']);
var submit = new NativeFunction(submitAddr, 'uint64',
                                ['pointer','int32','uint32','pointer','int32']);

var state = Memory.alloc(16);
state.add(0).writeS32(83);  state.add(4).writeS32(67);
state.add(8).writeS32(55);  state.add(12).writeS32(462);

var hallmark = reseal(state, 0x10);
var outbuf   = Memory.alloc(256);
var verdict  = submit(state, 0x10, hallmark, outbuf, 255);
console.log("verdict:", verdict, "cert:", outbuf.readCString());
```
 

![Frida hooks](/images/articles/htb-cyberapocalypse-mobile/17_pm_frida_hooks.png)

```text
[*] reseal_core @ 0x76b15acd0c10
    53 48 83 ec 50 0f 28 05      SH..P.(.        ← prologue matches
[*] submit_core @ 0x76b15acd10d0
    31 c0 83 fe 10 75 13 f3      1....u..        ← prologue matches
[+] Computed hallmark: 3695541232 (0xdc457bf0)
[+] Verdict: 1
```
 

Verdict moved from `0` (`REJECT_STATE`) to **`1` (`REJECT_TOKEN`)** \- i.e. _“perfect face, wrong spine.”_ The state is accepted as genuine; only the hallmark is wrong. This proves the extracted bytes are right, and proves `reseal()`’s output is **not** the hallmark `submit()` wants - the two gates are independent.

### Step 8 - Skip the hallmark entirely

We do not actually need the hallmark. The keystream generator is a small deterministic function of a single 32-bit intermediate `M`, and we know the plaintext starts with `HTB{`. A 2³² search is trivial in C.

First dump the 28-byte table at `DAT_001006a0`:

![Keystream table](/images/articles/htb-cyberapocalypse-mobile/16_pm_keystream_table.png)

```text
2a 53 db 7b a3 5d 34 f5 5f 59 74 5e 00 43 88 1c
a1 13 6f b7 f8 d7 3f 79 c1 b0 af 1a
```
 
```c
static const uint8_t table[28] = {
    0x2a,0x53,0xdb,0x7b,0xa3,0x5d,0x34,0xf5,0x5f,0x59,0x74,0x5e,0x00,0x43,
    0x88,0x1c,0xa1,0x13,0x6f,0xb7,0xf8,0xd7,0x3f,0x79,0xc1,0xb0,0xaf,0x1a };

#define K1 0x85ebca6bU
#define K2 0xc2b2ae35U
static inline uint32_t f1(uint32_t x){ return ((x >> 16) ^ x) * K1; }
static inline uint32_t f2(uint32_t x){ return ((x >> 13) ^ x) * K2; }
static inline uint32_t f3(uint32_t x){ return  (x >> 16) ^ x;      }

int try_seed(uint32_t M, uint8_t *out) {
    uint32_t s = M;
    for (int i = 0; i < 28; i++) {
        s = f1(s + K2);
        uint32_t y = f2(s);
        s = f3(y);
        out[i] = (uint8_t)(y >> 24) ^ table[i];
    }
    return out[0]=='H' && out[1]=='T' && out[2]=='B' && out[3]=='{';
}

for (uint64_t M = 0; M < (1ULL << 32); M++)
    if (try_seed((uint32_t)M, out)) printf("M=0x%08x -> %.28s\n", (uint32_t)M, out);
```
 
```bash
gcc -O3 -o solve solve_proofmark.c && ./solve
```
 

The matching seed prints the decrypted 28-byte certificate directly.

### Flag

```text
┌──(kali㉿DESKTOP-B2E2TUG)-[/mnt/c/Users/adm_consultant/Downloads/mobile_proofmark]
└─$ cat > solve_proofmark.c << 'EOF'
#include <stdio.h>
#include <stdint.h>

#define K1 0x85ebca6bU
#define K2 0xc2b2ae35U
#define ADD 0xc2b2ae35U

static inline uint32_t f1(uint32_t x) { return ((x >> 16) ^ x) * K1; }
static inline uint32_t f2(uint32_t x) { return ((x >> 13) ^ x) * K2; }
static inline uint32_t f3(uint32_t x) { return  (x >> 16) ^ x;      }

static const uint8_t table[28] = {
    0x2a,0x53,0xdb,0x7b,0xa3,0x5d,0x34,0xf5,
    0x5f,0x59,0x74,0x5e,0x00,0x43,0x88,0x1c,
    0xa1,0x13,0x6f,0xb7,0xf8,0xd7,0x3f,0x79,
    0xc1,0xb0,0xaf,0x1a
};

static int try_seed(uint32_t M, uint8_t *out) {
    uint32_t s = M;
    for (int i = 0; i < 28; i++) {
        s = f1(s + ADD);
        uint32_t y = f2(s);
        s = f3(y);
        out[i] = (uint8_t)(y >> 24) ^ table[i];
    }
    return out[0] == 'H' && out[1] == 'T' && out[2] == 'B' && out[3] == '{';
}

int main(void) {
    uint8_t out[29];
    uint64_t total = 1ULL << 32;
    printf("sweeping %llu seeds...\n", (unsigned long long)total);
    for (uint64_t m = 0; m < total; m++) {
        if (try_seed((uint32_t)m, out)) {
            out[28] = 0;
            printf("\n[+] M = 0x%08x\n", (uint32_t)m);
            printf("[+] plaintext: %s\n", out);
        }
        if ((m & 0x0FFFFFFF) == 0 && m)
            printf("  %.0f%%\n", (double)m / total * 100.0);
    }
    printf("done.\n");
    return 0;
}
EOF
gcc -O3 -o solve solve_proofmark.c && ./solve
sweeping 4294967296 seeds...
  6%

[+] M = 0x1c7c8990
[+] plaintext: HTB{p3rf3ct_f4c3_tru3_sp1n3}
  12%

[+] M = 0x2bd302a4
[+] plaintext: HTB{c��y�Zw护Ӗ�9�c���ſ?
  19%
  25%
  31%
  38%
  44%
  50%
  56%
  62%
  69%
  75%
  81%
  88%
  94%
done.

HTB{p3rf3ct_f4c3_tru3_sp1n3}        # emitted by solve_proofmark.c
```
 

* * *

* * *

## SaltCrown

### Summary

A crowd of the dead marches in lockstep to a city bell. You cannot fight them. Instead you wedge “shards” into narrow points along the street, timed to the bell’s rhythm, to break their step and overload the bell until its counterfeit strike-plate cracks.

![SaltCrown](/images/articles/htb-cyberapocalypse-mobile/20_saltcrown_launch.png)

On paper this is a timing/skill challenge. In practice, reading the code reveals two shortcuts:

  * The flag depends only on **which** narrow points you jammed - **not** on how well you timed them. The game re-looks-up the “correct” timing value at the end anyway.
  * Those “correct” timing values come from a compiled function whose only input is a **static 256-byte data file shipped inside the APK**. Nothing random, nothing runtime.

So you extract that file, reimplement the function offline, get all eight values, and then try the handful of possible jam-combinations until one decrypts to readable text. The game is never played.

* * *

## Proof of Concept (Technical)

### Step 1 - Identify the hybrid architecture

SaltCrown is **both** : C# game logic in `SaltCrown.dll`, _plus_ a native GDExtension.

```text
./SaltCrown/assets/.godot/mono/publish/x86_64/SaltCrown.dll
./SaltCrown/lib/x86_64/libashvault.android.template_release.x86_64.so
```
 
```ini
[configuration]
entry_symbol = "saltcrown_library_init"
compatibility_minimum = "4.7"
```
 

### Step 2 - Read the C# mechanism

`SaltCrownSpec` holds the sealed flag and the fold function (FNV-1a offset basis + FNV prime → this is FNV-1a with an extra avalanche):

```csharp
private static readonly byte[] SealedSpec = new byte[24]
{
    117, 201, 171, 107, 154,  83, 207, 191,  31, 233,
    126,  74, 147, 148,  37, 224,  41, 207, 135, 169,
    194, 128, 222, 220
};
public const uint Unmeasured = 2166136261u;      // FNV-1a offset basis

public static uint Measure(uint acc, int chokeIndex, uint phaseBucket)
{
    acc = Mix(acc, (uint)chokeIndex);
    acc = Mix(acc, phaseBucket);
    return acc;
}

public static string Unseal(uint measured)
{
    byte[] a = new byte[SealedSpec.Length];
    uint num = measured;
    for (int i = 0; i < SealedSpec.Length; i++) {
        num = Mix(num, (uint)i);
        a[i] = (byte)(SealedSpec[i] ^ (byte)(num >> 24));
    }
    return "HTB{" + Encoding.ASCII.GetString(a) + "}";
}

private static uint Mix(uint h, uint v)
{
    h ^= v;  h *= 16777619;          // FNV prime
    h ^= h >> 15; h *= 2246822519u;
    h ^= h >> 13;
    return h;
}
```
 

### Step 3 - Find what actually feeds `Measure`

`Director.Forge()` runs the moment the strike-plate fractures:

```csharp
private void Forge()
{
    uint num = 2166136261u;
    foreach (ShardSeat item in from s in _seats
                               where s.Bites
                               orderby s.ChokeIndex
                               select s)
    {
        num = SaltCrownSpec.Measure(num, item.ChokeIndex,
                                    Tolerances.PhaseBucket(item.ChokeIndex));
    }
    Crown = SaltCrownSpec.Unseal(num);
    State = Phase.Forged;
}
```
 

**This is the critical observation.** The second argument is `Tolerances.PhaseBucket(item.ChokeIndex)` \- freshly re-queried by index. It is **not** `item.BeddedPhase`, the phase you actually struck at. Your timing only decides _whether_ a seat counts (`s.Bites`); it contributes nothing to the hash.

Therefore: **the flag is a pure function of the set of choke indices you successfully seated.**

Timing gating lives in `Tolerances`, and is irrelevant to the output:

```csharp
public static bool EngagesCleanly(int index, float phaseAtBedding)
    => BucketDistance(BeatBucket(phaseAtBedding), PhaseBucket(index)) <= 8;

public static uint PhaseBucket(int index) => WearLattice.AdmitBucket(index);
```
 

And `WearLattice` is the bridge into native code:

```csharp
public const string RubbingPath = "res://rubbings/ashvault.dat";

public static uint AdmitBucket(int choke)
{
    if (_vault   == null) _vault   = ClassDB.Instantiate("AshVault").AsGodotObject();
    if (_rubbing == null) _rubbing = LoadRubbing();
    byte[] array = new byte[_rubbing.Length];
    Array.Copy(_rubbing, array, _rubbing.Length);
    return (uint)(int)_vault.Call("admit_bucket", array, choke);
}
```
 

Its only inputs are a **static asset file** and an integer 0–7. Fully reproducible offline.

### Step 4 - Locate `admit_bucket` in the native library

Tracing `saltcrown_library_init` shows the usual godot-cpp bootstrap:

![saltcrown_library_init](/images/articles/htb-cyberapocalypse-mobile/21_sc_libinit.png)

Rather than walk the whole registration chain, search the binary for the method name directly and follow its cross-reference:

![admit_bucket string](/images/articles/htb-cyberapocalypse-mobile/22_sc_admit_string.png)

This leads to **`FUN_0011dc30`** \- the implementation.

### Step 5 - Read the algorithm

```c
ulong FUN_0011dc30(undefined8 param_1, undefined8 param_2, int param_3)
{
  // ---- Stage 1: build a 64-word table from the rubbing bytes (FNV-1a variant) ----
  do {
    uVar5 = 0x811c9dc5;                                  // FNV-1a offset basis
    ...
      uVar5 = ((uint)*(byte *)(lVar2 + lVar7)     + iVar3 ^ uVar5) * 0x1000193;
      uVar5 = (uVar5 >> 0xb ^ (uint)*(byte *)(lVar2 + 1 + lVar7) + iVar3 ^ uVar5) * 0x1000193;
      uVar5 =  uVar5 >> 0xb ^ uVar5;
    ...
    local_228[lVar4] = uVar5;
  } while (lVar4 != 0x40);                               // 64 words
  ...
```
 

![Stage 2 + extraction](/images/articles/htb-cyberapocalypse-mobile/23_sc_algo_stage2.png)

```c
  // ---- Stage 2: 4096 rounds of neighbour-coupled mixing ----
  do {
      ...
      uVar5 = ((local_228[(uint)lVar4 & 0x3f] << 0x1b | local_228[(uint)lVar4 & 0x3f] >> 5) ^
               (local_228[(int)lVar2 - 1U & 0x3f] << 7 | local_228[(int)lVar2 - 1U & 0x3f] >> 0x19)
               ^ uVar6) * -0x7a143589 + iVar8 + (int)lVar2;
      uVar6 = (uVar6 << 0xd | uVar6 >> 0x13) ^ (uVar5 >> 0xf ^ uVar5) * -0x3d4d51c3;
      local_128[lVar2] = uVar6 >> 0x10 ^ uVar6;
      ...
    memcpy(local_228, local_128, 0x100);
    iVar8 = iVar8 + -0x61c88647;                         // XXTEA golden-ratio delta
  } while (iVar3 != 0x1000);                             // 4096 rounds

  // ---- Final: two indexed words → one byte ----
  uVar6 = (local_128[param_3 * 0x17 + 0x29U & 0x3f] << 0xb |
           local_128[param_3 * 0x17 + 0x29U & 0x3f] >> 0x15) ^ local_128[param_3 * 7 + 3U & 0x3f];
  return (ulong)((uVar6 >> 5 ^ uVar6 >> 0xd) & 0xff);
}
```
 

`0x61c88647` is the two’s-complement form of the XXTEA/golden-ratio delta `0x9E3779B9` \- this is an XXTEA-flavoured block permutation over 64 × 32-bit words.

### Step 6 - Cross-check against raw disassembly (this step matters)

A first Python port produced garbage. The decompiler’s C had folded a step away. Reading the raw loop body instead:

![Round body assembly](/images/articles/htb-cyberapocalypse-mobile/24_sc_asm_round.png)
    
    
    0011dd38  ROL   param_3,0x7                              ; rol(local_228[(i-1)&63], 7)
    0011dd3b  ROL   param_2+0x4,0x1b                         ; rol(local_228[(i+1)&63], 27)
    0011dd3e  XOR   param_2+0x4,param_3
    0011dd40  XOR   param_2+0x4,param_1+0x4                  ; ^= local_228[i]
    0011dd42  IMUL  param_3,param_2+0x4,-0x7a143589
    0011dd48  ADD   param_3,R12D                             ; += round delta
    0011dd4b  ADD   param_3,EAX                              ; += i
    0011dd4d  MOV   param_2+0x4,param_3
    0011dd4f  SHR   param_2+0x4,0xf
    0011dd52  XOR   param_2+0x4,param_3                      ; x ^= x >> 15
    0011dd54  ROL   param_1+0x4,0xd                          ; rol(local_228[i], 13)
    0011dd57  IMUL  param_3,param_2+0x4,-0x3d4d51c3
    0011dd5d  XOR   param_1+0x4,param_3
    

and then, immediately before the store:

![Hidden xorshift](/images/articles/htb-cyberapocalypse-mobile/25_sc_asm_xorshift.png)
    
    
    0011dd5f  MOV   param_3,param_1+0x4
    0011dd61  SHR   param_3,0x10
    0011dd64  XOR   param_3,param_1+0x4        ; <-- x ^= x >> 16   (missed on first pass)
    0011dd66  MOV   dword ptr [RSP + RAX*0x4 + 0x100],param_3
    

That final `x ^= x >> 16` avalanche is present in the machine code but easy to drop when transcribing from decompiled C. **Every one of 4096 × 64 iterations depends on it** \- omit it and the output is noise.

The tail extraction confirms the index math too:
    
    
    0011dd9b  LEA   EAX,[RBX*0x8]      ; 8c
    0011dda2  SUB   EAX,EBX            ; 7c
    0011dda4  ADD   EAX,0x3            ; 7c + 3
    0011dda7  LEA   ECX,[RBX + RBX*0x2]; 3c
    0011ddaa  SHL   ECX,0x3            ; 24c
    0011ddad  SUB   ECX,EBX            ; 23c
    0011ddaf  ADD   ECX,0x29           ; 23c + 41   (== 0x17*c + 0x29)
    0011ddbc  ROL   ECX,0xb
    0011ddc2  XOR   ECX,[... local_128[idx2]]
    0011ddcb  SHR   EAX,0xd
    0011ddce  SHR   ECX,0x5
    0011ddd1  XOR   ECX,EAX
    0011ddd3  MOVZX EAX,CL             ; & 0xff
    

### Step 7 - Extract the rubbing and reimplement offline

```bash
$ ls -la SaltCrown/assets/rubbings/ashvault.dat
-rwxrwxrwx 1 kali kali 256 ...  ashvault.dat

$ xxd SaltCrown/assets/rubbings/ashvault.dat | head -4
00000000: a75b 3f06 3f32 1130 ad51 1108 3514 072a  .[?.?2.0.Q..5..*
00000010: ae72 160f 3e23 3019 a458 2029 041d 0e03  .r..>#0..X )....
00000020: b559 2d24 2d30 0322 af43 331a 3736 1528  .Y-$-0.".C3.76.(
00000030: cc20 747d 6c41 427b f63a 427b 666f 6c51  . t}lAB{.:B{folQ
```
 
```python
def admit_bucket(rubbing: bytes, choke: int) -> int:
    size = len(rubbing)
    local_228 = [0] * 64

    # Stage 1 - FNV-1a variant, pairwise over the rubbing, seeded per word index
    for word_idx in range(64):
        h, p = 0x811c9dc5, 0
        limit = (size & MASK32) - (size & 1)
        while limit != p:
            h = ((((rubbing[p]     + word_idx) & MASK32) ^ h) * 0x1000193) & MASK32
            t = (h >> 0xb) ^ (((rubbing[p + 1] + word_idx) & MASK32) ^ h)
            h = (t * 0x1000193) & MASK32
            h = ((h >> 0xb) ^ h) & MASK32
            p += 2
        if size & 1:
            h = ((((rubbing[p] + word_idx) & MASK32) ^ h) * 0x1000193) & MASK32
            h = ((h >> 0xb) ^ h) & MASK32
        local_228[word_idx] = h

    # Stage 2 - 4096 rounds
    local_128 = [0] * 64
    delta = 0
    M1 = (0x100000000 - 0x7a143589) & MASK32
    M2 = (0x100000000 - 0x3d4d51c3) & MASK32
    for _ in range(0x1000):
        for i in range(64):
            cur = local_228[i]
            A = rol32(local_228[(i + 1) & 0x3f], 0x1b)
            B = rol32(local_228[(i - 1) & 0x3f], 0x07)
            x = (((A ^ B) ^ cur) * M1 + delta + i) & MASK32
            y = (rol32(cur, 0xd) ^ (((x >> 0xf) ^ x) * M2)) & MASK32
            local_128[i] = (y ^ (y >> 0x10)) & MASK32      # <-- the recovered step
        local_228 = local_128[:]
        delta = (delta - 0x61c88647) & MASK32

    v = rol32(local_128[(choke * 0x17 + 0x29) & 0x3f], 0xb) ^ local_128[(choke * 7 + 3) & 0x3f]
    return ((v >> 5) ^ (v >> 0xd)) & 0xff
```
 
```text
Rubbing file size: 256 bytes
choke 0 -> bucket 88
choke 1 -> bucket 43
choke 2 -> bucket 121
choke 3 -> bucket 112
choke 4 -> bucket 2
choke 5 -> bucket 240
choke 6 -> bucket 62
choke 7 -> bucket 188
```
 

### Step 8 - Search the choke combinations

`Elric.SeatStock = 5`, and `Director` folds biting seats **ordered by`ChokeIndex`**. That is a tiny search space, so enumerate it and keep whatever decrypts to printable ASCII:

```python
from itertools import combinations_with_replacement

def try_sequence(chokes, buckets):
    acc = 2166136261
    for c in chokes:                       # already ascending
        acc = measure(acc, c, buckets[c])
    return unseal(acc)

for k in range(0, 9):
    for combo in combinations_with_replacement(range(8), k):
        out = try_sequence(combo, buckets)
        if all(32 <= b < 127 for b in out):
            print(combo, "HTB{" + out.decode() + "}")
```
 

Here is the full script:

```text
#!/usr/bin/env python3
"""
Offline solver for SaltCrown's admit_bucket() native function.
Usage: python3 solve_saltcrown.py /path/to/ashvault.dat
"""
import sys

MASK32 = 0xFFFFFFFF

def rol32(x, n):
    x &= MASK32
    n &= 31
    if n == 0:
        return x
    return ((x << n) | (x >> (32 - n))) & MASK32

def admit_bucket(rubbing: bytes, choke: int) -> int:
    size = len(rubbing)
    if size < 1:
        return 0xFFFFFFFFFFFFFFFF

    local_228 = [0] * 64

    # Stage 1: build local_228[64] via FNV1a-like pairwise mixing over the rubbing bytes
    for word_idx in range(64):
        uVar5 = 0x811c9dc5
        lVar7 = 0
        iVar3 = word_idx
        if size != 1:
            limit = (size & MASK32) - (size & 1)
            while limit != lVar7:
                b0 = rubbing[lVar7]
                b1 = rubbing[lVar7 + 1]
                uVar5 = (((b0 + iVar3) & MASK32) ^ uVar5)
                uVar5 = (uVar5 * 0x1000193) & MASK32
                term = (uVar5 >> 0xb) ^ (((b1 + iVar3) & MASK32) ^ uVar5)
                uVar5 = (term * 0x1000193) & MASK32
                uVar5 = ((uVar5 >> 0xb) ^ uVar5) & MASK32
                lVar7 += 2
        if (size & 1) != 0:
            b0 = rubbing[lVar7]
            uVar5 = ((((b0 + iVar3) & MASK32) ^ uVar5) * 0x1000193) & MASK32
            uVar5 = ((uVar5 >> 0xb) ^ uVar5) & MASK32
        local_228[word_idx] = uVar5

    # Stage 2: 4096 rounds of mixing (XXTEA-golden-ratio-constant style)
    local_128 = [0] * 64
    iVar8 = 0
    MULT1 = (0x100000000 - 0x7a143589) & MASK32  # unsigned form of -0x7a143589
    MULT2 = (0x100000000 - 0x3d4d51c3) & MASK32  # unsigned form of -0x3d4d51c3

    for _round in range(0x1000):
        lVar2 = 0
        while True:
            lVar4 = lVar2 + 1
            uVar6 = local_228[lVar2]

            idxA = lVar4 & 0x3f
            A = (rol32(local_228[idxA], 0x1b) | (local_228[idxA] >> 5)) & MASK32

            idxB = (lVar2 - 1) & 0x3f
            B = (rol32(local_228[idxB], 7) | (local_228[idxB] >> 0x19)) & MASK32

            uVar5 = ((A ^ B) ^ uVar6) & MASK32
            uVar5 = (uVar5 * MULT1) & MASK32
            uVar5 = (uVar5 + iVar8 + lVar2) & MASK32

            rotated = rol32(uVar6, 0xd)
            mixpart = ((uVar5 >> 0xf) ^ uVar5) & MASK32
            uVar6new = (rotated ^ ((mixpart * MULT2) & MASK32)) & MASK32

            local_128[lVar2] = uVar6new

            lVar2 = lVar4
            if lVar4 == 0x40:
                break

        local_228 = local_128[:]
        iVar8 = (iVar8 - 0x61c88647) & MASK32

    idx1 = (choke * 0x17 + 0x29) & 0x3f
    idx2 = (choke * 7 + 3) & 0x3f
    val1 = local_128[idx1]
    uVar6 = (rol32(val1, 0xb) | (val1 >> 0x15)) & MASK32
    uVar6 ^= local_128[idx2]
    uVar1 = (uVar6 >> 5) ^ (uVar6 >> 0xd)
    return uVar1 & 0xff

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 solve_saltcrown.py /path/to/ashvault.dat")
        sys.exit(1)

    with open(sys.argv[1], "rb") as f:
        rub = f.read()

    print(f"Rubbing file size: {len(rub)} bytes")
    buckets = []
    for choke in range(8):
        b = admit_bucket(rub, choke)
        buckets.append(b)
        print(f"choke {choke} -> bucket {b}")

    print()
    print("All buckets:", buckets)
    
```
 

### Flag

```text
┌──(kali㉿DESKTOP-B2E2TUG)-[/mnt/c/Users/adm_consultant/Downloads/mobile_salt_crown]
└─$ python3 solve_saltcrown.py "SaltCrown/assets/rubbings/ashvault.dat"
Rubbing file size: 256 bytes
choke 0 -> bucket 149
choke 1 -> bucket 84
choke 2 -> bucket 104
choke 3 -> bucket 178
choke 4 -> bucket 26
choke 5 -> bucket 6
choke 6 -> bucket 101
choke 7 -> bucket 234

Brute-forcing choke sequences (0-8 biting seats, repeats allowed)...
Checked 12870 combinations. Top 15 by printable-byte score:

score=24/24 chokes=(3, 4, 5, 6, 7) -> HTB{p3rf3ct_f4c3_wr0ng_sp1n3}
score=17/24 chokes=(0, 0, 4, 4, 6) -> HTB{yj3��r@<SP<L=dqCT>@���}
score=17/24 chokes=(5, 5, 5, 5, 5, 7) -> HTB{V�J�jf��>I~G�twI�@T/�:4X}
score=17/24 chokes=(0, 1, 2, 3, 6, 6, 6) -> HTB{,d_/a'��_��in Z+��c2u%�o}
score=17/24 chokes=(0, 1, 2, 6, 6, 6, 7) -> HTB{k0�d8(�IK��zv un�r*!@��Y}
score=17/24 chokes=(1, 1, 3, 5, 5, 6, 6) -> HTB{D3o�\��H'Lzv��2-�Frd5Ux}
score=17/24 chokes=(3, 4, 4, 4, 6, 6, 7) -> HTB{:4Y_,��>U��Y)Yv:p]�+My}
score=17/24 chokes=(0, 0, 2, 4, 5, 5, 5, 7) -> HTB{�C]�+S� I�z/UKVbZfs� 4}
score=17/24 chokes=(0, 1, 2, 4, 4, 6, 7, 7) -> HTB{�=J�dVwRu {Lwh��iw��s1n�}
score=17/24 chokes=(0, 2, 2, 2, 6, 7, 7, 7) -> HTB{<��cu[-@d[q25�db_g   %�U}
oJgREd��lA�5��biM]�.|<}2, 2, 4, 4, 7) -> HTB{�
score=16/24 chokes=(0, 0, 0, 1, 2, 4, 5) -> HTB0ou��P\H8Y��iv�d�jm[HB}
score=16/24 chokes=(0, 1, 3, 4, 4, 4, 5) -> HTB{+�ck��iV~&�d��hA.+U;{�v}
score=16/24 chokes=(0, 2, 5, 5, 6, 7, 7) -> HTB{~|\Z�(u�VHIt�q6�r�5DF}
score=16/24 chokes=(1, 1, 3, 3, 3, 4, 7) -> HTB{g��E�,v�q~�y,CGp���O.?Mw}

HTB{p3rf3ct_f4c3_wr0ng_sp1n3}       # emitted by solve_saltcrown.py
```
 

* * *

# Scripts used

File | Challenge | Purpose  
---|---|---  
`scripts/invert_splitmix64.py` | Overstrike | Inverts splitmix64 to recover `CarriedMark` from `TrueSeal`  
`scripts/unseal_overstrike.py` | Overstrike | Reimplements `UnsealRegistry()` offline to print the flag  
`scripts/frida_proofmark.js` | Proofmark | Calls `reseal`/`submit` inside the live process to validate the extracted state  
`scripts/solve_proofmark.c` | Proofmark | 2³² keystream-seed search anchored on the `HTB{` prefix  
`scripts/solve_saltcrown.py` | SaltCrown | Offline `admit_bucket` reimplementation + choke-combination search  
  
**Tooling:** apktool · ILSpy · GDRE Tools · Ghidra · Frida · gcc/python3

* * *

# Takeaways

  * **All three were solvable without playing the games.** Overstrike and SaltCrown never required the emulator at all; Proofmark used it only to _validate_ a hypothesis before committing to an offline brute-force.
  * **Find the check, not the gameplay.** In every case the win condition was a pure function of a small number of inputs, and the “skill” framing was a decoy - most explicitly in SaltCrown, where `Forge()` discards your actual timing and re-derives the expected value by index.
  * **Recognise stock constants.** `0x9E3779B9`, `0x85EBCA6B`, `0xC2B2AE35`, `0x01000193`, `0x811C9DC5` instantly identify splitmix64, MurmurHash3 and FNV-1a, which collapses “unknown custom crypto” into “known, invertible, or cheaply searchable.”
  * **Trust the disassembly over the decompiler.** The single hardest bug in the whole set was one `x ^= x >> 16` that Ghidra’s C reconstruction folded away. Decompiled C is a summary, not the program.
  * **Brute-force the small unknown.** Twice the winning move was refusing to reverse an expensive function (1.2M rounds; 4096 rounds) and instead searching the tiny space around it - a 32-bit seed, or a few thousand index combinations.
