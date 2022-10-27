---
published: false
---
---
title: Barclay's Red Team Entry Challenge (Delta)
author: Lime
date: 2022-10-26 14:10:00 +0800
categories: [Writeup, Web, RE]
tags: [ctf, reversing]
render_with_liquid: false
---
## Overview
The machine didn't require any footholds into the actual box itself. It held a easy web challenege requiring no enumeration apart from active recon. The reversing aspect was a easy/low medium, with the hardest parts being leveraging the decryption function of the .net console application.

## Path to Flag

The first thing I did was run passive scans on the given IP and look at the certificate. The certificate was a standard "LetsEncrypt"  had nothing meaningful. I did more DNS recon and nothing was to be found. 

I also did some basic service enumeration and the box had ports tls/80 (http) and tls/443 (https) open with. The latter webpage being the start of the challenge. 

![Screen Shot 2022-08-17 at 12.55.04 AM.png](/_posts/Screen Shot 2022-08-17 at 12.55.04 AM.png)


The webpage has 2 sets of coordinates that lead to  to a hidden sub directory and inputted them into a website.  The  `chesters.zip` sub directory exists with this identifier next too `52.1915314,-2.2186969`. 

![Screen Shot 2022-08-17 at 1.08.18 AM.png](/_posts/Screen Shot 2022-08-17 at 1.08.18 AM.png)

![Screen Shot 2022-08-17 at 1.13.23 AM.png](/_posts/Screen Shot 2022-08-17 at 1.13.23 AM.png)


Тhe zip file is password protected with the password being `icecream`.


Browsing through the zip file there are no hidden files. There are two directories; `reversing` and `web`.  Reversing contains the main loot while note.txt contains aother hidden directory. I didn't use it, so it was probably a rabbit hole. 

![Screen Shot 2022-08-17 at 1.46.08 AM.png](/_posts/Screen Shot 2022-08-17 at 1.46.08 AM.png)


The flag however is legitmate with `Crypt.exe` being a regulat dotnet console application . We can determine this by using Detect it Easy (https://github.com/horsicq/Detect-It-Easy).

![Screen Shot 2022-08-17 at 3.16.44 AM.png](/_posts/Screen Shot 2022-08-17 at 3.16.44 AM.png)

When running the PE it prompts us to enter a Password. I tried guessing and got a Incorrect Password screen when I did.

I then threw it into ghidra and noticed some curious strings.  

![Screen Shot 2022-08-17 at 3.18.02 AM.png](/_posts/Screen Shot 2022-08-17 at 3.18.02 AM.png)

To my dismay we couldn't retrieve any passwords from here. I then put it into ILSpy (https://github.com/icsharpcode/AvaloniaILSpy) to better decompile the assembely.  Looking at the Main function we can see how the program works more acutely. 

![Screen Shot 2022-08-17 at 3.22.49 AM.png](/_posts/Screen Shot 2022-08-17 at 3.22.49 AM.png)

The password is stored in base 64 encoded XOR or for a visual (Base 64(XOR(Password))).  Our entered password is XOR'd and then Base64'd and compared to the stored encoded password string and if they equal we are granted access. 

One important thing is that the string `EC4F2208-9E0A-491F-B8E5-813118127C37` is the key that XOR uses to encrypt. so we have to use that in our decryption input and specify our key is `UTF8`.

![Screen Shot 2022-08-17 at 3.29.28 AM.png](/_posts/Screen Shot 2022-08-17 at 3.29.28 AM.png)

We can check if this works by inputting our new decoded string into the application. 


![Screen Shot 2022-08-17 at 3.32.18 AM.png](/_posts/Screen Shot 2022-08-17 at 3.32.18 AM.png)

 Our Password checks out which leads to the next function which is the encrypt function.  I put some random characters in and observed my output.

![Screen Shot 2022-08-17 at 3.34.15 AM.png](/_posts/Screen Shot 2022-08-17 at 3.34.15 AM.png)

The encryption matches the same as the encoded flag text whith the identifier being the odd colon in the middle.

This I decided to look at the `public static Crypt.Crypto.Crypto` to see how the encryption works in hopes of being able to RE it and decrypt the flag.

![Screen Shot 2022-08-17 at 3.38.54 AM.png](/_posts/Screen Shot 2022-08-17 at 3.38.54 AM.png)

To our luck there is already a decrypt function making our job easier.  It looks like the string has 2 parts. the first being the RSA enryption then a colon splitting the second part which is AES encryption.  It does this by creating a string named `text` which contains the output of `System.Guid.NewGuid().ToString()` which creates a unique identifier according to (https://social.msdn.microsoft.com/Forums/en-US/3a8ef576-e02c-4ae5-b073-a4826a2d10d5/systemguidnewguidtostring?forum=aspdatasourcecontrols)

![Screen Shot 2022-08-17 at 3.50.08 AM.png]({{site.baseurl}}/_posts/Screen Shot 2022-08-17 at 3.50.08 AM.png)


Whith `ToString()` being our identifer of `"d"` Im fairly certain at least.  I'll explain a redundancy I placed just in case I was wrong later. It then encrypts that string with AES. Then our RSA encryption comes in and encrypts our inputted string using this function.

![Screen Shot 2022-08-17 at 3.54.58 AM.png](/_posts/Screen Shot 2022-08-17 at 3.54.58 AM.png)

Which retrieves the public key from `Resources` which is names `s1` It then shoots us back RSA encrypted text.  One thing to note is that the RSA decryption function exists in the same area calling `s2` from `Resources` which is the private key. A colon is placed in between and our AES text is then added.

We can use the built in decrypt function thats already written in to decrypt our encoded strings and consequently retrieve the flag. 

Using ILSpy's download code option I exported the bulk majority of the code and create a dotnet console project in Visual Studio and dumped the code and then cleaned it up a bit.

![Screen Shot 2022-08-17 at 4.02.13 AM.png](/_posts/Screen Shot 2022-08-17 at 4.02.13 AM.png)

There are a couple things we have to do before we can run this code though however.  We have to edit our project file.

![Screen Shot 2022-08-17 at 4.05.38 AM.png]({{site.baseurl}}/_posts/Screen Shot 2022-08-17 at 4.05.38 AM.png)

Add `<LangVersion>8.0</LangVersion>` because whenever I tried running the code it ran under C# version 7.3 for some reason. Another thing while we are here is my redundancy. I wasn't so sure in my theroy about the GUID function so I added this GUID that I found in ghidra while looking for string to my project file in lieu of the one it gave me just in case.

![Screen Shot 2022-08-17 at 4.07.18 AM.png](/_posts/Screen Shot 2022-08-17 at 4.07.18 AM.png)

From here we can start editing this source code to add the already present decryption functions.  In the Main class I added my own code that decrypts what you input into the application.
``` C#

namespace Crypt
{
	internal class Program
	{
		private static void Main(string[] args)
		{
			Console.WriteLine("Enter password to proceed: ");
			string input = Console.ReadLine();
			if (base64.Base64Encode(xor.xorIt("EC4F2208-9E0A-491F-B8E5-813118127C37", input)) != "MSx7Zl9TfmENagBzM2hgShA=")
			{
				Console.WriteLine("Unauthorized access detected!");
				Thread.Sleep(10000);
				return;
			}
			Console.WriteLine("Authentication successful.");
			Console.WriteLine("LIME Decrypt XD :");
			Console.WriteLine(Crypt.Crypto.Crypto.Decrypt(Console.ReadLine())); // This is the Decryption
			Console.WriteLine("Done. Press any key to terminate.");
			Console.ReadLine();
		}
	}
```

There are a couple other things we have to do before we run this application just to be safe. We have to add the refrences to our project that exist in the PE too. We can see them in ILSpy btw under the `Refrences`. 

![Screen Shot 2022-08-17 at 4.15.22 AM.png](/_posts/Screen Shot 2022-08-17 at 4.15.22 AM.png)


Now that we got that we can proceed to the last step. Which is to find a way to call the Private key in the code.  This is done near the bottom with this function.  

``` c#
public static string Decryption(string strText)
		{
			string @string = Resources.ResourceManager.GetString("s2");// Gets the private key from Resources using ResourceManager
			Encoding.UTF8.GetBytes(strText);
			using RSACryptoServiceProvider rSACryptoServiceProvider = new RSACryptoServiceProvider(1024);
			try
			{
				rSACryptoServiceProvider.FromXmlString(@string);
				byte[] rgb = Convert.FromBase64String(strText);
				byte[] bytes = rSACryptoServiceProvider.Decrypt(rgb, fOAEP: true);
				return Encoding.UTF8.GetString(bytes).ToString();
			}
			finally
			{
				rSACryptoServiceProvider.PersistKeyInCsp = false;
			}
		}
	}
```

The strings are stored under `Crypt.Properties.Resources.resources` . Looking at `ResourceManager` closer we can see whats going on more fully.  These two docs explain the process inately:  1(https://docs.microsoft.com/en-us/dotnet/api/system.resources.resourcemanager?view=net-6.0) & 2(https://docs.microsoft.com/en-us/dotnet/core/extensions/create-resource-files)

I went exhausted myself trying to create my own resource file and string table to add the RSA keys, but ran into some troubles since I was using Mac and VSCode wasn't outling step to do that correctly using `rsgen.exe` or its equvilant on Mac unfortunately.

So I decided to just remove the `Resources.ResourceManager.GetString("s2")`  and just add the private key without any retreival.  My code looks a little like this.

``` c#
public static string Decryption(string strText)
        {
			string @string = ("<RSAKeyValue><Modulus>21wEnTU+mcD2w0Lfo1Gv4rtcSWsQJQTNa6gio05AOkV/Er9w3Y13Ddo5wGtjJ19402S71HUeN0vbKILLJdRSES5MHSdJPSVrOqdrll/vLXxDxWs/U0UT1c8u6k/Ogx9hTtZxYwoeYqdhDblof3E75d9n2F0Zvf6iTb4cI7j6fMs=</Modulus><Exponent>AQAB</Exponent><P>/aULPE6jd5IkwtWXmReyMUhmI/nfwfkQSyl7tsg2PKdpcxk4mpPZUdEQhHQLvE84w2DhTyYkPHCtq/mMKE3MHw==</P><Q>3WV46X9Arg2l9cxb67KVlNVXyCqc/w+LWt/tbhLJvV2xCF/0rWKPsBJ9MC6cquaqNPxWWEav8RAVbmmGrJt51Q==</Q><DP>8TuZFgBMpBoQcGUoS2goB4st6aVq1FcG0hVgHhUI0GMAfYFNPmbDV3cY2IBt8Oj/uYJYhyhlaj5YTqmGTYbATQ==</DP><DQ>FIoVbZQgrAUYIHWVEYi/187zFd7eMct/Yi7kGBImJStMATrluDAspGkStCWe4zwDDmdam1XzfKnBUzz3AYxrAQ==</DQ><InverseQ>QPU3Tmt8nznSgYZ+5jUo9E0SfjiTu435ihANiHqqjasaUNvOHKumqzuBZ8NRtkUhS6dsOEb8A2ODvy7KswUxyA==</InverseQ><D>cgoRoAUpSVfHMdYXW9nA3dfX75dIamZnwPtFHq80ttagbIe4ToYYCcyUz5NElhiNQSESgS5uCgNWqWXt5PnPu4XmCXx6utco1UVH8HGLahzbAnSy6Cj3iUIQ7Gj+9gQ7PkC434HTtHazmxVgIR5l56ZjoQ8yGNCPZnsdYEmhJWk=</D></RSAKeyValue>");
			Encoding.UTF8.GetBytes(strText);
			using RSACryptoServiceProvider rSACryptoServiceProvider = new RSACryptoServiceProvider(1024);
			try
			{
				rSACryptoServiceProvider.FromXmlString(@string);
				byte[] rgb = Convert.FromBase64String(strText);
				byte[] bytes = rSACryptoServiceProvider.Decrypt(rgb, fOAEP: true);
				return Encoding.UTF8.GetString(bytes).ToString();
			}
			finally
			{
				rSACryptoServiceProvider.PersistKeyInCsp = false;
			}
		}
	}
}

```

When we run this modifed code directly in VSCode and finally decode our flag :)
![Screen Shot 2022-08-17 at 4.33.16 AM.png](/_posts/Screen Shot 2022-08-17 at 4.33.16 AM.png)

