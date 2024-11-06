+++
date = '2024-11-06T14:16:59+07:00'
draft = false
title = 'CryptoCTF 2023 Writeup'
tags = ["Cryptography", "Writeup"]
# author = ["d4rk19ht"]
+++


## 1.Insights
```python
#!/usr/bin/env sage

from Crypto.Util.number import *
from flag import flag

def getRandomNBits(n):
	nb = '1' + ''.join([str(randint(0, 1)) for _ in range(n - 1)])
	return nb

def getLeader(L, n):
	nb = L + getRandomNBits(n)
	return int(nb, 2)

def genPrime(L, nbit):
	l = len(L)
	assert nbit >= l
	while True:
		p = getLeader(L, nbit - l)
		if is_prime(p):
			return p

def genKey(L, nbit):
	p, q = [genPrime(L, nbit) for _ in '__']
	n = p * q
	d = next_prime(pow(n, 0.2919))
	phi = (p - 1) * (q - 1)
	e = inverse(d, phi)
	pubkey, privkey = (n, e), (p, q)
	return pubkey, privkey

def encrypt(msg, pubkey):
	n, e = pubkey
	m = bytes_to_long(msg)
	c = pow(m, e, n)
	return c

nbit = 1024
L = bin(bytes_to_long(b'Practical'))[2:]
pubkey, privkey = genKey(L, nbit)
p, q = privkey
c = encrypt(flag, pubkey)

print('Information:')
print('-' * 85)
print(f'n = {p * q}')
print(f'e = {pubkey[1]}')
print(f'c = {c}')
print(f'p = {bin(p)[2:len(L)]}...[REDACTED]')
print(f'q = {bin(q)[2:len(L)]}...[REDACTED]')
print('-' * 85)
```
RSA challenge, d is know, so just find d and recover the flag
```python
n = 12765231982257032754070342601068819788671760506321816381988340379929052646067454855779362773785313297204165444163623633335057895252608396010414744222572161530653104640020689896882490979790275711854268113058363186249545193245142912930804650114934761299016468156185416083682476142929968501395899099376750415294540156026131156551291971922076435528869024742993840057342092865203064721826362149723366381892539617642364692012936270150691803063945919154346756726869466855557344213050973081755499746750276623648407677639812809665472258655462846021403503851719008687214848550916999977775070011121527941755954255781343103086789
e = 459650454686946706615371845737527916539205656667844780634386049268800615782964920944229084502752167395446158290854047696006034750210758341744841762479191173017773034647739346927390580848998121830029134542880713409306092967282675122699586503684943407535067216738556403169403622104762516293879994387324370835718056251706150557820106296417750402984941838652433642298378976899556042987560946508887315484380807248331504458640857234708123277403252632993828101306072382329879857946191508782246793011691530554606521701055094223574951862129713872918021549814674387049788995785872980320871421550616327471735316980754238323013
c = 10992248752412909788626396175372747713079469256270100576886987393986576680666320383209810005318254336440105142571546847427454822405793626080251363454531982746373841267986148332456716023293306870382809568309620264499225135226626560298741596462262513921032733814032790312163314776421380481083058518893602887082464123177575742160690315666730642727773288362853901330620841098230284739614618790097180848133698381487679399364400048499041582830157094876815030301231505774900176910650887780842536610942820066913075027528705150102760422836458745949063992228680293226303245265232017738712226154128654682937687199768621565945171
d = next_prime(pow(n, 0.2919))

from Crypto.Util.number import long_to_bytes as ltb
print(ltb(pow(int(c),int(d),int(n))))
```
FLAG: ```CCTF{RSA_N3w_rEc0rd5_4Nd_nEw_!nSi9h75!}```
## 2.Suction
```python
#!/usr/bin/env python3

from Crypto.Util.number import *
from flag import flag

def keygen(nbit, r):
	while True:
		p, q = [getPrime(nbit) for _ in '__']
		e, n = getPrime(16), p * q
		phi = (p - 1) * (q - 1)
		if GCD(e, phi) == 1:
			N = bin(n)[2:-r]
			E = bin(e)[2:-r]
			PKEY = N + E
			pkey = (n, e)
			return PKEY, pkey

def encrypt(msg, pkey, r):
	m = bytes_to_long(msg)
	n, e = pkey
	c = pow(m, e, n)
	C = bin(c)[2:-r]
	return C

r, nbit = 8, 128
PKEY, pkey = keygen(nbit, r)
print(f'PKEY = {int(PKEY, 2)}')
FLAG = flag.lstrip(b'CCTF{').rstrip(b'}')
enc = encrypt(FLAG, pkey, r)
print(f'enc = {int(enc, 2)}')
```
Base on the challenge, we do not know the last 8-bit of ```encrypted flag```, ```n``` and ```e```, so the simpliest way is to bruteforce all possible states, it will took ```256^3``` times to find the correct value. In here, I do a small optimization: I first find the n and e value, then if I found the right public key then I will start bruteforce encrypted flag. In the finding public key phase, if current ```n``` is divived by a small number (between range [2, 100000]), then remove it from possible n. As n only 256 bits long, so you just throw it in (factordb)[http://factordb.com/] and then it will do the rest. Finding e just the same, only chose the prime, remove number that is not prime from current list. Then the total of bruteforcing need is very small.
Source:
```python
PKEY = 55208723145458976481271800608918815438075571763947979755496510859604544396672
ENC = 127194641882350916936065994389482700479720132804140137082316257506737630761


N = int(bin(PKEY)[2:][:-8], 2)
E = int(bin(PKEY)[2:][-8:], 2)

from Crypto.Util.number import isPrime

possible_e = []
for i in range(0, (1<<8)):
    e = (E << 8 ) | i
    if isPrime(e):
        possible_e.append(e)

def small_lmao(n):
    for i in range(2, 100000):
        if n % i == 0:
            return False
    return True
# possible_n = []
# for i in range(0, (1<<8)):
#     n = (int(N) << int(8)) | int(i)
#     if small_lmao(n):
#         possible_n.append(n)

# for i in possible_n:
#     print(i)

p = 188473222069998143349386719941755726311
q = 292926085409388790329114797826820624883
n = p * q 
def printable(ok):
    for i in ok:
        if i > 127 :
            return False
    return True 
from Crypto.Util.number import long_to_bytes as ltb 

for i in range((1<<8)):
    enc = (int(ENC) << int(8)) | i 
    for e  in possible_e:
        d = pow(e, -1, (p-1)*(q-1))
        dec = ltb(pow(int(enc), int(d), int(n)))
        if printable(dec):
            print(dec)

```
FLAG: ```CCTF{6oRYGy&Dc$G2ZS}```
## 3.Resuction
```python
#!/usr/bin/env python3

from Crypto.Util.number import *
from flag import flag

from decimal import *

def keygen(nbit, r):
	while True:
		p, q = [getPrime(nbit) for _ in '__']
		d, n = getPrime(64), p * q
		phi = (p - 1) * (q - 1)
		if GCD(d, phi) == 1:
			e = inverse(d, phi)
			N = bin(n)[2:-r]
			E = bin(e)[2:-r]
			PKEY = N + E
			pkey = (n, e)
			return PKEY, pkey

def encrypt(msg, pkey, r):
	m = bytes_to_long(msg)
	n, e = pkey
	c = pow(m, e, n)
	C = bin(c)[2:-r]
	return C

r, nbit = 8, 1024
PKEY, pkey = keygen(nbit, r)
print(f'PKEY = {int(PKEY, 2)}')
FLAG = flag.lstrip(b'CCTF{').rstrip(b'}')
enc = encrypt(FLAG, pkey, r)
print(f'enc = {int(enc, 2)}')
```
This challenge is similar to the above, with a little bit upgrade.  This time, ```e``` are very big, but instead ```d``` are small (which if you play crypto, you probably know what attack I will use later). I do the same trick that I use in the previous challenge - only keep possible numbers that can be the public key, remove the other. In this attack, to recover ```d```, I use the Wiener attack (this apply in the case that ```d``` is less than ```n^(1/4)```)
Source:
```python
PKEY = 14192646310719975031517528381795548241077678859071194396837281472399230674325587198691913743313024193030641258581477544395982020385534616950314446352119543012689979705497443206671093873748633162188213109347667494028713308821945628880987033909436936504594085029207071182583896573425433818693447573712242776054326253393149643818428222532313129014785625379928796322492111783102063504659053965652092334907431265629283336869752591405010801363428649651892548988084920295512198406822149854508798413366425646089176325412867633899847841343293434518769693835679828109184625256833518392375615524221729773443578746961887429674099018040291053535429314874943299587513900900515776980848746070077651676814430155460898107362207677739452859298842563030631706907437662807884529549746843462830493900480079096937402325307522965877069080418178692616875205678928420840955518031258855869218152431304423803589723140983606576549207164114711076498723237274262054605174412193097533550076687418481230734706280737017543741247718967059747548710091320650704988384742281590019869955579949961574733610565593105027342454880292474482792325237942870408329807427182014062811782475262070063065860763705715879581562335668163076088547827008755841277828137570366416095778
ENC = 93313196155732054514477836642637636744872135106456047659057794344503071105783322399713135615723000054324693644981340568454628360027598469597864407205974007198804288563284521413279406211756674451582156555173212403946908121125010635246043311803494356106191509999360722019559999844621905376368824621365540442906142224342650371557766313381899279520110833822291649001754956653102495882094754863493058001964760438040783400782352466943990226083197340594364084294954324101604417550048379969516185353765224920719355485680872367930581872987972421836853197205534334204586713387469939986387582911728909524428102693874671302382


N = int(bin(PKEY)[2:][:2048 - 8], 2)

from Crypto.Util.number import isPrime

# possible_e = []
# for i in range(0, (1<<8)):
#     e = (E << 8 ) | i
#     if isPrime(e):
#         possible_e.append(e)

def small_lmao(n):
    for i in range(2, 1000000):
        if n % i == 0:
            return False
    return True
# possible_n = []
# for i in range(0, (1<<8)):
#     n = (int(N) << int(8)) | int(i)
#     if small_lmao(n):
#         possible_n.append(n)

# for i in possible_n:
#     print(i)


def cf_expansion(a,b):
    ls = []
    ls.append(a//b)
    a,b = b, a%b
    while(b!=0):
        ls.append(a // b)
        a , b = b, a%b
    return ls
import math
def cf_convergent(ls):
    n = []
    d = []
    for i in range(len(ls)):
        if (i == 0):
            n.append(ls[i])
            d.append(1)
        elif i == 1:
            n.append(ls[1]*ls[0] + 1)
            d.append(ls[1])
        else:
            n.append(n[i-1]*ls[i] + n[i-2])
            d.append(d[i-1]*ls[i] + d[i-2])
        yield n[i],d[i]

def solve(B,P):
    delta = B*B - 4*P
    if (delta<0):
        return 0,0
    rdel = math.isqrt(delta)
    return (B + rdel)//2, (B - rdel)//2

def find_d(n,e):
    for k,d in cf_convergent(cf_expansion(e,n)):
        if (k==0):
            continue
        phin = (e*d - 1)//k
        sum = n - phin + 1
        p, q = solve(sum, n)
        if (p*q==n):
            return d
    return 0

E = int(bin(PKEY)[2:][2048 - 8 : ], 2)
# for testn in possible_n:
#     for i in range(1<<8):
#         e = (int(E) << 8) | int(i)
#         d = find_d(testn, e)
#         if d != 0 and isPrime(int(d)):
#             print("Found pair: ")
#             print(f"n = {testn}")
#             print(f"e = {e}")
#             print(f"d = {d}")

n = 28781418259071163834545208786492597316357138268450456443121779857237190669654679502925616925907115061139426651454246296829614929839091896732956124186768298711851015827257060255218333952539548249210858753648965921585289379414151961197198777686222970660319202167442420274437451557166736926361972983650143650097981777542950972139757680517744639660898696901009088978971506526002932830312595664154921938706240176536981793499426543601513874115451315768319593051858239793153849410530285884330866972048864103208648273010126369559341912163849839663249252300813799486995834473605326584986843653735963725697383056972744506296271
e = 19152712448778858582528734875468196678366984818842265525346340114296810907435357813959451387293270496095878944786775775749129832803842508074794234774568097809721690831345120778762600713106116293626590641716601095020202233532544196547654794913903350183891867544539554967347396716482565232986995497267273877597593761608770699282404807896050347585632153075234094034163801474316224123620090879021107631960008144066862084573910442635526649884938724881478713853362879412453150893601267748827792136092224063120914443976032390554506925020506643629449426005820918745312311984391868895996772772355715765028825561022860823765675
d = 10254340117704573547

from Crypto.Util.number import long_to_bytes as ltb

def printable(ok):
    for i in ok:
        if i > 127 :
            return False
    return True 

for i in range(1<<8):
    enc = (int(ENC) << 8) | int(i)
    dec = ltb(pow(int(enc), int(d), int(n)))
    if printable(dec):
        print(dec)
        exit(0)
```
FLAG: ```CCTF{aIr_pr3s5urE_d!Ff3rEn7i4L_8eTw3eN_ArEa5!}```
## 4.Blue office
```python
#!/usr/bin/enc python3

import binascii
from secret import seed, flag

def gen_seed(s):
	i, j, k = 0, len(s), 0
	while i < j:
		k = k + ord(s[i])
		i += 1
	i = 0
	while i < j:
		if (i % 2) != 0:
			k = k - (ord(s[i]) * (j - i + 1))            
		else:
			k = k + (ord(s[i]) * (j - i + 1))
	
		k = k % 2147483647
		i += 1

	k = (k * j) % 2147483647
	return k

def reseed(s):
	return s * 214013 + 2531011

def encrypt(s, msg):
	assert s <= 2**32
	c, d = 0, s
	enc, l = b'', len(msg)
	while c < l:
		d = reseed(d)
		enc += (msg[c] ^ ((d >> 16) & 0xff)).to_bytes(1, 'big')
		c += 1
	return enc

enc = encrypt(seed, flag)
print(f'enc = {binascii.hexlify(enc)}')
```
An LCG base challenge, but in the reseed function, you may notice that it does not mod with something. Because of this, all we need to concern is the lower bits(or Least Significant Bits), as in this line ```msg[c] ^ ((d >> 16) & 0xff))```, only lower bits are affect the result. 

So my idea is, use the property above to bruteforce seed, then use that seed to calculate the flag
Source:
```python
enc = bytes.fromhex("b0cb631639f8a5ab20ff7385926383f89a71bbc4ed2d57142e05f39d434fce")

def reseed(s):
	return s * 214013 + 2531011


possible_seed = []

def check(seed):
	sample = [ord("C") ^ enc[0], ord("C") ^ enc[1], ord("T") ^ enc[2], ord("F") ^ enc[3], ord("{") ^ enc[4]][1:]
	okseed = ((ord("C") ^ enc[0]) << 16) | seed 
	for i in range(len(sample)):
		okseed = reseed(okseed)
		if ((okseed>>16) & 0xff) != sample[i]:
			return False
	return True

possible_seed = []
for testseed in range(1<<16):
	if check(testseed) :
		possible_seed.append(testseed)

print(possible_seed)
okseed = ((ord("C") ^ enc[0]) << 16) | possible_seed[0]

flag = "C"
for i in range(1, len(enc)):
	okseed = reseed(okseed)
	flag += chr(((okseed>>16) & 0xff) ^ enc[i])
	
print(flag)
```
FLAG: ```CCTF{__B4ck_0r!F1c3__C1pHeR_!!}```

## 5.Roldy
```python
#!/usr/bin/env python3

from Crypto.Util.number import *
from pyope.ope import OPE as enc
from pyope.ope import ValueRange
import sys
from secret import key, flag

def die(*args):
	pr(*args)
	quit()

def pr(*args):
	s = " ".join(map(str, args))
	sys.stdout.write(s + "\n")
	sys.stdout.flush()

def sc(): 
	return sys.stdin.buffer.readline()

def encrypt(msg, key, params):
	if len(msg) % 16 != 0:
		msg += (16 - len(msg) % 16) * b'*'
	p, k1, k2 = params
	msg = [msg[_*16:_*16 + 16] for _ in range(len(msg) // 16)]
	m = [bytes_to_long(_) for _ in msg]
	inra = ValueRange(0, 2**128)
	oura = ValueRange(k1 + 1, k2 * p + 1)
	_enc = enc(key, in_range = inra, out_range = oura)
	C = [_enc.encrypt(_) for _ in m]
	return C

def main():
	border = "|"
	pr(border*72)
	pr(border, " Welcome to Roldy combat, we implemented an encryption method to    ", border)
	pr(border, " protect our secret. Please note that there is a flaw in our method ", border)
	pr(border, " Can you examine it and get the flag?                               ", border)
	pr(border*72)

	pr(border, 'Generating parameters, please wait...')
	p, k1, k2 = [getPrime(129)] + [getPrime(64) for _ in '__']
	C = encrypt(flag, key, (p, k1, k2))

	while True:
			pr("| Options: \n|\t[E]ncrypted flag! \n|\t[T]ry encryption \n|\t[Q]uit")
			ans = sc().decode().lower().strip()
			if ans == 'e':
				pr(border, f'encrypt(flag, key, params) = {C}')
			elif ans == 't':
				pr(border, 'Please send your message to encrypt: ')
				msg = sc().rstrip(b'\n')
				if len(msg) > 64:
					pr(border, 'Your message is too long! Sorry!!')
				C = encrypt(msg, key, (p, k1, k2))
				pr(border, f'C = {C}')
			elif ans == 'q':
				die(border, "Quitting ...")
			else:
				die(border, "Bye ...")

if __name__ == '__main__':
	main()
```
This time, we are facing an oracle challenge. The server give us 2 choice:
-	Get the encrypted flag
-	Encrypt an message

So in the first glance, I notice that the server used the pyOPE cryptosystem. After searching, I know that pyOPE is an implementation of Boldyreva symmetric order-preserving encryption scheme. This scheme has an interesting property:
```
if m1 < m2, then f(m1) < f(m2)
```
Because of this, we can use binary search to searching for the correct flag. In my solution, you will notice that I reconnect to the server multiple times. This happened because server doesn't allow  to send mupltiple queries (I think it's for keep the server away from crashing), but of course this doesn't affect the solution much.

Source:
```python
from pwn import *
from Crypto.Util.number import long_to_bytes as ltb, bytes_to_long as btl 
s = remote("02.cr.yp.toc.tf", 31377)

def establish():
    global s
    s = remote("02.cr.yp.toc.tf", 31377)
    s.recvuntil(b"[Q]uit\n")
    s.sendline(b"e")

    s.recvuntil(b" = ")
    enc = eval(s.recvline(0).decode())
    return enc 


def encrypt(val):
    global s
    s.sendlineafter(b"[Q]uit\n", b"t")
    s.sendlineafter(b"encrypt: \n", ltb(val))
    s.recvuntil(b" = ")
    ok = eval(s.recvline(0).decode())
    return ok[0]


def bs(blockid):
 #   [89370465111452328101028130174938390560, 90554308795198268800333083338000572447]
    flag = b""
    s.close()
    for i in range(len(flag), 16):
        l = 0
        r = 127
        sleep(1)
        enc = establish()
        while l < r:
            testchar = ((l + r) // 2) + 1
            print(f"chartest: {testchar}")
            payload = flag + bytes([testchar]) + b"\x00" * (16 - i - 1)
            ok = encrypt(btl(payload))
            if ok <= enc[blockid]:
                l = testchar
            else:
                r = testchar - 1
        flag += bytes([l])

        print(f"Flag found: {flag}")
        s.close()
    return flag

flag = b""
flag += bs(0)
flag += bs(1)
flag += bs(2)
flag += bs(3)
print(flag)
```

FLAG: ```CCTF{Boldyreva_5ymMe7rIC_0rD3r_pRe5Erv!n9_3nCryp7i0n!_LStfig9TM}```