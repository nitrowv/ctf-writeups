# The Vault

**Writeup by:** nitrowv   
**Category:** Reversing   
**Difficulty:** Medium    


We're given the binary `vault`. After opening it in Ghidra, we start at the main function and after going through an intermediate function, reach the following:
```C
void FUN_0010c220(void)

{
  bool bVar1;
  byte bVar2;
  long in_FS_OFFSET;
  byte local_241;
  uint local_234;
  char local_219;
  basic_ifstream<char,std--char_traits<char>> local_218 [520];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  basic_ifstream((char *)local_218,0x10e004);
  bVar2 = is_open();
  if ((bVar2 & 1) == 0) {
    operator<<<std--char_traits<char>>((basic_ostream *)&cout,"Could not find credentials\n");
                    /* WARNING: Subroutine does not return */
    exit(-1);
  }
  bVar1 = true;
  local_234 = 0;
  while( true ) {
    local_241 = 0;
    if (local_234 < 0x19) {
      local_241 = good();
    }
    if ((local_241 & 1) == 0) break;
    get((char *)local_218);
    bVar2 = (***(code ***)(&PTR_PTR_00117880)[(byte)(&DAT_0010e090)[(int)local_234]])();
    if ((int)local_219 != (uint)bVar2) {
      bVar1 = false;
    }
    local_234 = local_234 + 1;
  }
  if (bVar1) {
    operator<<<std--char_traits<char>>
              ((basic_ostream *)&cout,"Credentials Accepted! Vault Unlocking...\n");
  }
  else {
    operator<<<std--char_traits<char>>
              ((basic_ostream *)&cout,
               "Incorrect Credentials - Anti Intruder Sequence Activated...\n");
  }
  ~basic_ifstream(local_218);
  if (*(long *)(in_FS_OFFSET + 0x28) == local_10) {
    return;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

We see that the program is looking for `flag.txt` and that the contents of this file will go through some checks and the vault will open. The main check occurs in the following `while( true )` loop:
```C
  bVar1 = true;
  local_234 = 0;
  while( true ) {
    local_241 = 0;
    if (local_234 < 0x19) {
      local_241 = good();
    }
    if ((local_241 & 1) == 0) break;
    get((char *)local_218);
    bVar2 = (***(code ***)(&PTR_PTR_00117880)[(byte)(&DAT_0010e090)[(int)local_234]])();
    if ((int)local_219 != (uint)bVar2) {
      bVar1 = false;
    }
    local_234 = local_234 + 1;
  }
```
Variable `local_234` represents the position in the text file, going up to `0x19 == 25`. A check is performed on this character and if it is incorrect, `bVar1`, the variable that determines acceptance, is false. The check is as follows:
```C
bVar2 = (***(code ***)(&PTR_PTR_00117880)[(byte)(&DAT_0010e090)[(int)local_234]])();
```

Opening the program in Hopper makes this part of the code a bit easier to understand (at least for me).
```C
            do {
                    local_241 = 0x0;
                    if (sign_extend_64(local_234) < 0x19) {
                            var_23A = std::basic_ios<char, std::char_traits<char> >::good();
                            local_241 = var_23A;
                    }
                    if ((local_241 & 0x1) == 0x0) {
                        break;
                    }
                    rax = std::istream::get(&local_218);
                    rsi = *(int8_t *)(sign_extend_64(local_234) + 0x10e090) & 0xff;
                    rdi = *(0x117880 + rsi * 0x8);
                    rcx = *rdi;
                    rcx = *rcx;
                    bVar2 = (rcx)(rdi, rsi, 0x8, rcx);
                    if (sign_extend_64(local_219) != (bVar2 & 0xff)) {
                            bVar1 = 0x0;
                    }
                    local_234 = local_234 + 0x1;
                }
```

`rsi` is the value at the address  `(local_234 + 0x10e090) & 0xff`, which leads us to:

```
        0010e090 e0              ??         E0h
        0010e091 d1              ??         D1h
        0010e092 bb              ??         BBh
        0010e093 27              ??         27h    '
        0010e094 f6              ??         F6h
        0010e095 72              ??         72h    r
        0010e096 db              ??         DBh
        0010e097 a3              ??         A3h
        0010e098 83              ??         83h
        0010e099 b9              ??         B9h
        0010e09a 69              ??         69h    i
        0010e09b 23              ??         23h    #
        0010e09c db              ??         DBh
        0010e09d 63              ??         63h    c
        0010e09e b9              ??         B9h
        0010e09f 23              ??         23h    #
        0010e0a0 05              ??         05h
        0010e0a1 2b              ??         2Bh    +
        0010e0a2 2b              ??         2Bh    +
        0010e0a3 83              ??         83h
        0010e0a4 23              ??         23h    #
        0010e0a5 39              ??         39h    9
        0010e0a6 45              ??         45h    E
        0010e0a7 39              ??         39h    9
        0010e0a8 92              ??         92h
```
For the first character, the value we're after for `rsi` is `0xe0`. From here, we have to perform `rdi = *(0x117880 + rsi * 0x8)` for each value. I performed these calculations with the Windows 10 built-in Programmer calculator. For brevity, I will only show this for the first character.


For the first character, we would do `0x117f80 = (0x117880 + 0xe0 * 0x8)`, resulting in `rdi = *(0x117f80)`, which leads to:
```
                             PTR_FUN_001152a8                                XREF[1]:     00117780(*)  
001152a8 60 d2 10        addr       FUN_0010d260
         00 00 00 
         00 00
```

Following this, we reach this function which returns the character for the iteration, which in this case is `H`.
```C
undefined8 FUN_0010d260(void)

{
  return 0x48;
}
```

After painstakingly calculating all of the characters and following the functions, we get the flag:

 `HTB{vt4bl3s_4r3_c00l_huh}`
