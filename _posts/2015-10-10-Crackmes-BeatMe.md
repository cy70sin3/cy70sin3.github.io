---
layout: post
title: Crackmes - BeatMe
---


> hello !  
the task is simple :)  
just try to make a working keygen for this crackme !  
good luck and have a good day :)

<a href="http://www.crackmes.de/users/rezk2ll/beatme/">Crackmes - BeatMe</a>

![Screenshot]({{ site.url }}/assets/crackmes-BeatMe/file.png)
So the task seems simple enough, given a 32-Bit ELF file, write a keygen. Sweet let's do it.

{% highlight text %}
.text:08048080                 public start
.text:08048080 start           proc near
.text:08048080                 call    sub_8048132
.text:08048085                 mov     esi, offset aVtfsobnf ; "VTFSOBNF!;!"
.text:0804808A                 call    sub_80481D3
{% endhighlight %}

Straight up we can see what looks like an encoded string pointed to by esi being passed to the function at 0x080481D3.

{% highlight text %}
.text:080481D3 sub_80481D3     proc near               ; CODE XREF: start+Ap
.text:080481D3                                         ; start+5Ep ...
.text:080481D3                 mov     edi, esi
.text:080481D5
.text:080481D5 loc_80481D5:                            ; CODE XREF: sub_80481D3+Aj
.text:080481D5                 lodsb
.text:080481D6                 test    al, al
.text:080481D8                 jz      short loc_80481DF
.text:080481DA                 dec     al
.text:080481DC                 stosb
.text:080481DD                 jmp     short loc_80481D5
{% endhighlight %}

This is looking like a basic Ceaser Cipher with the character pointed to by esi shifted at 0x080481DA. 

{% highlight c %}
  v1 = sys_read(0, &user_80493EC, 0x14u);       // Read in username
  JUMPOUT(v1, 3, &loc_80482A0);
  if ( v1 >= 10 )
  {
    byte_8049421 = 1;
    JUMPOUT(&loc_80482A0);
  }
  user_byte_804941E = v1 - 1;                   // Store username
  sub_80481D3(aQbttxpse);
  v2 = sys_write(1, aQbttxpse, 0xCu);
  v3 = sys_read(0, &pass_804940A, 0xCu);        // Read in password
  JUMPOUT(v3, 5, &loc_80482A0);
  if ( v3 >= 12 )
  {
    byte_8049421 = 1;
    JUMPOUT(&loc_80482A0);
  }
  pass_byte_804941F = v3 - 1;                   // Store password
  checks_sub_80481E4();                         // Additional checks
}
{% endhighlight %}  

Moving onto user input we can see that the username is stored at 0x080493EC and the password stored at 0x0804940A.  
Basic length checks are performed and the values are then moved to 0x0804941E and 0x0804941F respectively.  
Finally it calls the function at 0x080481E4 which performs additional checks.

{% highlight c %}
  if ( user_byte_804941E == pass_804940A[0] - 48)// pass[0] == len(user)
  {
    v0 = __rdtsc();
    dword_80493E6 = v0;
    v1 = __rdtsc();
    if ( (signed int)v1 - dword_80493E6 <= 13568 )
    {
      if ( pass_804940A[1] == byte_80493EE ) // pass[1] == user[2]
      {
        sub_80481D3(&pass_804940A[2]); // decrypt(pass[2:])
        v2 = &pass_804940A[2];
        v3 = &pass_804940A[2];
        for ( i = (unsigned __int8)user_byte_804941E >> 1; ; *v3++ = v5 - i ) // shift
        {
          v5 = *v2++;
          if ( !v5 )
            break;
        }
        v6 = (unsigned __int8)user_byte_804941E;
        *v3 = 0;
        sub_80481E1(v6);
        v9 = &pass_804940A[2];
        v10 = &user_80493EC;
        do
        {
          if ( !v8 )
            break;
          v7 = *v9++ == *v10++; // pass[2:] == user
          --v8;
        }
        while ( v7 );
        if ( v7 && *(v9 - 2) == *(v10 - 2) ) // win
        {
          sub_80481D3(aDpssfduZpvXjo);
          v11 = sys_write(1, aDpssfduZpvXjo, 0x14u);
          goto LABEL_17;
        }
      }
    }
{% endhighlight %}

The additional checks are simple enough, first it checks that pass[0] == len(user) and pass[1] == user[2].  
pass[2:] is then decoded with the same Ceaser Cipher as before and an additional shift is applied.  
If the resulting password is equal to the username the program will print 'CORRECT , YOU WIN'.

![Screenshot]({{ site.url }}/assets/crackmes-BeatMe/win.png)

## Solution

{% highlight python linenos %}

def encrypt(user):
  password = str(len(user)) + user[2]
  for i in user:
    password += chr(ord(i)+1+len(user)/2)
  print password

user = raw_input('Username: ')
encrypt(user)

{% endhighlight %}
-----
