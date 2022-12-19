---
title: Rust Reversing
author: nautilus
date: 2022-12-19 00:00:00 
categories: [Writeups,Reversing]
tags: [rust,ghidra,BackdoorCTF22]
---
This post shows how to reverse a rust binary (non stripped) using ghidra. The challege file can be found [here](https://github.com/0xn4utilus/backdoor22) along with the source code.

# Reversing a simple rust binary using ghidra

## 5p4gh3tt1 | BackdoorCTF22

Loading the binary in Ghidra, we can see that in the `_start` there is a `main` function as usually is the case with non-stripped binaries.

![1.png](/assets/img/20221219/5p4gh3tt1/1.png)

Inside the `main` function we can see the reference to the `chall` namespace.
![2.png](/assets/img/20221219/5p4gh3tt1/2.png)

We can view the namespaces in ghidra from `Symbol Tree`.

![4.png](/assets/img/20221219/5p4gh3tt1/4.png){: height="600" style="max-width: 30%" }

Inside `chall` we can find all the functions.

![5.png](/assets/img/20221219/5p4gh3tt1/5.png)

The `RECIPE` namespace is due to the use of `Lazy<Mutex>`, you can read more about it [here](https://docs.rs/once_cell/latest/once_cell/sync/struct.Lazy.html).

```rust

static RECIPE: Lazy<Mutex<String>> = Lazy::new(|| Mutex::new(String::new()));
fn update_recipe(value: String) {
    *RECIPE.lock().unwrap() = value;
}
fn get_recipe() -> String {
    RECIPE.lock().unwrap().to_string()
}
```

## Understanding the binary

We have to input the recipe which is also the flag. `update_recipe` sets our entered recipe int the `RECIPE` mutex we saw above, and `get_recipe` fetches the recipe from there.
    ![6.png](/assets/img/20221219/5p4gh3tt1/6.png)

One thing you will notice here is that unlike c/c++ binaries where we can directly see the string values, in rust we can't directly do that. Here we can see there are a lot of byte variables like `in_stack_ffffffffffffff67`, well all the strings (and not just strings) are stored somewhere in stack and are of fixed length, thus we can see a lot of `CONCAT`s. So it is a little tedious to find the strings but you can find the refrences and know what string(precisely what data) is being referred to.
  
In `new_recipe`, there are `update_recipe` and `get_recipe` functions, and there is a check on the length of recipe. Input must be of length 0x28 or 40.
    ![7.png](/assets/img/20221219/5p4gh3tt1/7.png)

We can see the program checks for three different functions that are `boiling_spaghetti`,`chopping_veggies` and `garnishing` if all are true, entered flag is correct. Let's solve the flag part by part.
    ![8.png](/assets/img/20221219/5p4gh3tt1/8.png)

This is the first part, you can easily understand the logic and functions like `add_oil`, `add_water` all just are appling `xor` and `addition` operations on ascii values of  first 10 characters, and are easy to reverse.
![9.png](/assets/img/20221219/5p4gh3tt1/9.png)

Now we have the first part of the flag.

```python
vec = [0xb7,0x29,0x1af,0x72,0x207,0xfa,0x167,0xd1,0xc1,0x13]

def oil_rev(x):
    return (((((x-9)^0x44)-0x31)^0x4f)-3)

def water_rev(x):
    return (x^0x4d)-0x58

def salt_rev(x):
    return (x-6)//4

def pepper_rev(x):
    return ((x^5)-6)^0x38

# for i in range(10):
#     if i%2==0:
#         print(chr(max([0,water_rev(oil_rev(vec[i]))])) ,chr(max([0,salt_rev(water_rev(vec[i]))])))
#     else:
#         print(chr(max([0,oil_rev(pepper_rev(vec[i])) ] )) ,chr(max([0,oil_rev(salt_rev(vec[i]))])))

part1="flag{m34tb"
```

Also, if you are unclear what a function in rust does, refer to the official documentaion [here](https://doc.rust-lang.org/).
    ![10.png](/assets/img/20221219/5p4gh3tt1/10.png)

In part 2, there is a `chop_veggies` function, which has two parameters first it the `ascii_chr` (veggie) and the other is `quantity`, quantitiy is calculated from the characters of part 1 of flag.  
    ![13.png](/assets/img/20221219/5p4gh3tt1/13.png)
    ![12.png](/assets/img/20221219/5p4gh3tt1/12.png)

```python

# part-2
y=[0x28e00, 0x9925, 0x5a9e, 0x4fb5, 0x85a8, 0xbcec, 0xa56e,0x17b55, 0x4f35, 0x4fb5,0x18c41, 0xbcec,0x2da80, 0x92c0, 0x2a74, 0xae74, 0xa3e0, 0xc11a, 0x8edf,0x1ec25]

quant1,quant2,quant3 = 0,0,0
for i in part1:
    if ord(i)%3==0:
        quant1+=ord(i)
    elif ord(i)%3==1:
        quant2 +=ord(i)
    else:
        quant3 += ord(i)

# print(quant1,quant2,quant3)
def chopping_veggies_rev():
    part2=""
    for i in range(20):
        if i%3==0:
            part2+=chop_veggies_rev(y[i],quant1)
        elif i%3==1:
            part2+=chop_veggies_rev(y[i],quant2)
        else:
            part2+=chop_veggies_rev(y[i],quant3)
    return part2

def chop_veggies_rev(x,quant):
    alpha_numeric = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_"
    for ch in alpha_numeric:    
        if ord(ch)%3==0:
            if (ord(ch)*quant)^quant == x: 
                return ch
        elif ord(ch)%3==1:
            if (ord(ch)^quant)*quant == x: 
                return ch
        else:
            if (ord(ch)*quant)^ord(ch) == x: 
                return ch

part2 = chopping_veggies_rev()
```

The third part is the easiest, the ascii va    ![14.png](/assets/img/20221219/5p4gh3tt1/14.png)lues of characters of last 10 characters are substituted with a special character, and the resulting string is reversed.
    ![11.png](/assets/img/20221219/5p4gh3tt1/11.png)
    ![14.png](/assets/img/20221219/5p4gh3tt1/14.png)

```python 
# part-3

# "%@!&*/+#.)"
part3 = ''.join(map(chr, [0x7d,0x48,0x34,0x33,0x32,0x30,0x4c,0x42,0x41,0x35] ))[::-1]

flag = part1+part2+part3
print(flag)
```

And we have our flag `flag{m34tb4ll5_4nd_5p4gh3tt1_45ABL0234H}`.

---

I would suggest going through [this](https://rustc-dev-guide.rust-lang.org/part-2-intro.html), if you want a better understanding of Rust Compiler. 

P.S.
Rust has the best compiler :P.

![meme.png](/assets/img/20221219/5p4gh3tt1/meme.png){: style="max-width: 40%" }