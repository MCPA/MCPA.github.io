---
layout: post
title: "Toddler's Bottle [fd]"
category: wargame pwnable
author: Leander
---
<strong>Description:</strong> [fd] is a 1 point challenge hosted on the online wargame site [pwnable.kr](http://pwnable.kr). The challenge involves understanding file descriptors and a little bit of C code. A login is required to be able to access the challenges because the site keeps score between competitiors that are registered and undergoing challenges.
<!--break-->

When first accessing the challenge we are provided the following information:
<figure class="highlight"><pre><code class="language-text" data-lang="text">
Mommy! what is a file descriptor in Linux?

ssh fd@pwnable.kr -p 2222 (pw:guest)</code></pre></figure>

## At the beginning

So now we access the challenge through connecting via ssh and then we are provided three files.

![initial entry](/images/fd_initial_entry.png)

Next let's examine the source code. First we can see on line 8 the program expect to receive at least one argument and that it expects it to be a number. Next on line 12 a variable called ```fd``` is assigned a value based on <em>argv[1]</em>. Next on line 14, the program reads up to 32 characters from the variable ```fd```, places it into another variable ```buf```. Between lines 16 and and 19 the program compares the value in buf to ```"LETMEWIN\n"```. Last it either gives us the flag or gives us a hint that we need to ```learn about Linux file IO\n"```

{% highlight c linenos %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char buf[32];

int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;}
	
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);}
	
	printf("learn about Linux file IO\n");
return 0;}{% endhighlight %}

## Making mistakes and not paying enough attention

To begin with I thought that I needed to pass the program a hexidecimal value that would equal ```LETMEWIN\n``` and make sure that it was at least <em>0x1234</em> greater than my input. For example: I converted all the ascii to hex using ```python -c 'import binascii; print binascii.hexlify("LETMEWIN\n");'``` which produced ```4c45544d4557494e0a``` and then I added 0x1234 manually.

## Solving the problem

After about 40-60 minutes of frustration I realized that this was a file descriptor problem and that really what I needed was to use the <strong><em>read()</em></strong>[^1] on line 14 to take advantage of this.

In linux/unix operations systems there are generally three standard file descriptors that are always predefined <em>0, 1, and 2</em>.[^2] After realizing this,  I simply calculated the decimal value for 0x1234, which is 4660 and added 1. When I input the value 4661 it set in motion the solution to get the flag.

Here is how it worked:

1. Execute the program with the value we calculated, ```./fd 4661```
2. The progam validates that it has at least one argument at line 8.
3. Line 12 converts our ascii input into an integer; subtracts the hex value 0x1234 (also 4660 in decimal); and then assigns the value to the integer variable ```fd```.
4. Line 14 then executes a read of fd, which now reads from ```stdin``` (remember the value of fd is 1 now?)
5. The program is now waiting for input, so now we ```type in LETMEWIN and enter```. Keep in mind when we hit enter it will automatically send the <em>"\n"</em> that we need.
6. Problem solved and now we got our flag!

Here's how it looks when you take the correct steps - notice I left off the flag. The real practice is going and doing this for yourself!
![fd solution](/images/fd_solution.png)

<h3>Footnotes:</h3>
[^1]: <strong><em>Read()</strong></em> is a standard C library function. it's prototype looks like: ```read(int fildes, void *buf, size_t nbyte)```. Read() attempts to read nbyte bytes of data from the object referenced by the descriptor fildes into the buffer pointed to by buf.
[^2]: File descriptor 0 is <em>stdin</em>; file descriptor 1 is <em>stdout</em>; and file descriptor 2 is <em>stderr</em>. Go [here](http://www.gnu.org/software/libc/manual/html_node/Streams-and-File-Descriptors.html) to learn about streams and file descriptors. You can also learn about standard file descriptors provided by the linux/unix operations systems [here](http://www.bottomupcs.com/file_descriptors.html).