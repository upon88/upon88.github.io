---
title: 查看CRT会话中的密文
date: 2021-12-08 17:28:48
tags:
---
# 环境
env:(windows10,SecureCRT8.1，python39)
# 步骤
## 1、github上下载[how-does-SecureCRT-encrypt-password](https://github.com/upon88/how-does-SecureCRT-encrypt-password)工具
一个python文件 SecureCRTCipher.py ，以及如何使用详细说明，我在后面贴上代码：
参数介绍：
``` 
Usage:
    SecureCRTCipher.py <enc|dec> [-v2] [-p ConfigPassphrase] <plaintext|ciphertext>

    <enc|dec>              "enc" for encryption, "dec" for decryption.
                           This parameter must be specified.（加密|解密）必选项

    [-v2]                  Encrypt/Decrypt with "Password V2" algorithm.
                           This parameter is optional.（如果加密使用的是Password V2算法则加上这个参数）可选项

    [-p ConfigPassphrase]  The config passphrase that SecureCRT uses.
                           This parameter is optional.（如果你的SecureCRT打开时要密码，则要加上这个参数，并在后面加上你使用的密码）可选项

    <plaintext|ciphertext> Plaintext string or ciphertext string.
                           NOTICE: Ciphertext string must be a hex string.
                           This parameter must be specified.（明文或密文，密文必须是16进制的字符串）必选项
``` 
## 2、[安装Python](https://www.python.org/downloads/windows/)
## 3、安装pycryptodome模块
```
pip3 install pycryptodome
```
## 4、找到SecureCRT会话配置文件
Options->Global Options->Category->Configuration Paths->Configuration folder文件夹下的Sessions文件夹下对应的会话配置文件xxx.ini，并将其打开

## 5、在SecureCRTCipher.py文件目录下cmd命令行窗口中执行命令即可查看明文：
### 如果内容像这样：
```
......S:"Username"=root
S:"Password"=
S:"Password V2"=02:7b9f594a1f39bb36bbaa0d9688ee38b3d233c67b338e20e2113f2ba4d328b6fc8c804e3c02324b1eaad57a5b96ac1fc5cc1ae0ee2930e6af2e5e644a28ebe3fc
S:"Login Script V2"=
```
则执行以下指令查看：
```
python SecureCRTCipher.py dec -v2 7b9f594a1f39bb36bbaa0d9688ee38b3d233c67b338e20e2113f2ba4d328b6fc8c804e3c02324b1eaad57a5b96ac1fc5cc1ae0ee2930e6af2e5e644a28ebe3fc
```
### 如果内容像这样：
```
...
...
S:"Username"=root
D:"[SSH2] Port"=00000016
S:"Password"=uc71bd1c86f3b804e42432f53247c50d9287f410c7e59166969acab69daa6eaadbe15c0c54c0e076e945a6d82f9e13df2
D:"Session Password Saved"=00000001
```
则执行以下指令查看：（命令行中字符串比Password中的字符串少个u）
```bash
python SecureCRTCipher.py dec c71bd1c86f3b804e42432f53247c50d9287f410c7e59166969acab69daa6eaadbe15c0c54c0e076e945a6d82f9e13df2
```

注意：参数一定要和自己配置匹配，像我一开始没加-p参数就报错。Error: Failed to encrypt.

SecureCRTCipher.py文件内容如下：
```python
#!/usr/bin/env python3
import os
from Crypto.Hash import SHA256
from Crypto.Cipher import AES, Blowfish

class SecureCRTCrypto:

    def __init__(self):
        '''
        Initialize SecureCRTCrypto object.
        '''
        self.IV = b'\x00' * Blowfish.block_size
        self.Key1 = b'\x24\xA6\x3D\xDE\x5B\xD3\xB3\x82\x9C\x7E\x06\xF4\x08\x16\xAA\x07'
        self.Key2 = b'\x5F\xB0\x45\xA2\x94\x17\xD9\x16\xC6\xC6\xA2\xFF\x06\x41\x82\xB7'

    def Encrypt(self, Plaintext : str):
        '''
        Encrypt plaintext and return corresponding ciphertext.

        Args:
            Plaintext: A string that will be encrypted.

        Returns:
            Hexlified ciphertext string.
        '''
        plain_bytes = Plaintext.encode('utf-16-le')
        plain_bytes += b'\x00\x00'
        padded_plain_bytes = plain_bytes + os.urandom(Blowfish.block_size - len(plain_bytes) % Blowfish.block_size)

        cipher1 = Blowfish.new(self.Key1, Blowfish.MODE_CBC, iv = self.IV)
        cipher2 = Blowfish.new(self.Key2, Blowfish.MODE_CBC, iv = self.IV)
        return cipher1.encrypt(os.urandom(4) + cipher2.encrypt(padded_plain_bytes) + os.urandom(4)).hex()

    def Decrypt(self, Ciphertext : str):
        '''
        Decrypt ciphertext and return corresponding plaintext.

        Args:
            Ciphertext: A hex string that will be decrypted.

        Returns:
            Plaintext string.
        '''

        cipher1 = Blowfish.new(self.Key1, Blowfish.MODE_CBC, iv = self.IV)
        cipher2 = Blowfish.new(self.Key2, Blowfish.MODE_CBC, iv = self.IV)
        ciphered_bytes = bytes.fromhex(Ciphertext)
        if len(ciphered_bytes) <= 8:
            raise ValueError('Invalid Ciphertext.')
        
        padded_plain_bytes = cipher2.decrypt(cipher1.decrypt(ciphered_bytes)[4:-4])
        
        i = 0
        for i in range(0, len(padded_plain_bytes), 2):
            if padded_plain_bytes[i] == 0 and padded_plain_bytes[i + 1] == 0:
                break
        plain_bytes = padded_plain_bytes[0:i]

        try:
            return plain_bytes.decode('utf-16-le')
        except UnicodeDecodeError:
            raise(ValueError('Invalid Ciphertext.'))

class SecureCRTCryptoV2:

    def __init__(self, ConfigPassphrase : str = ''):
        '''
        Initialize SecureCRTCryptoV2 object.

        Args:
            ConfigPassphrase: The config passphrase that SecureCRT uses. Leave it empty if config passphrase is not set.
        '''
        self.IV = b'\x00' * AES.block_size
        self.Key = SHA256.new(ConfigPassphrase.encode('utf-8')).digest()

    def Encrypt(self, Plaintext : str):
        '''
        Encrypt plaintext and return corresponding ciphertext.

        Args:
            Plaintext: A string that will be encrypted.

        Returns:
            Hexlified ciphertext string.
        '''
        plain_bytes = Plaintext.encode('utf-8')
        if len(plain_bytes) > 0xffffffff:
            raise OverflowError('Plaintext is too long.')
        
        plain_bytes = \
            len(plain_bytes).to_bytes(4, 'little') + \
            plain_bytes + \
            SHA256.new(plain_bytes).digest()
        padded_plain_bytes = \
            plain_bytes + \
            os.urandom(AES.block_size - len(plain_bytes) % AES.block_size)
        cipher = AES.new(self.Key, AES.MODE_CBC, iv = self.IV)
        return cipher.encrypt(padded_plain_bytes).hex()

    def Decrypt(self, Ciphertext : str):
        '''
        Decrypt ciphertext and return corresponding plaintext.

        Args:
            Ciphertext: A hex string that will be decrypted.

        Returns:
            Plaintext string.
        '''
        cipher = AES.new(self.Key, AES.MODE_CBC, iv = self.IV)
        padded_plain_bytes = cipher.decrypt(bytes.fromhex(Ciphertext))
        
        plain_bytes_length = int.from_bytes(padded_plain_bytes[0:4], 'little')
        plain_bytes = padded_plain_bytes[4:4 + plain_bytes_length]
        if len(plain_bytes) != plain_bytes_length:
            raise ValueError('Invalid Ciphertext.')

        plain_bytes_digest = padded_plain_bytes[4 + plain_bytes_length:4 + plain_bytes_length + SHA256.digest_size]
        if len(plain_bytes_digest) != SHA256.digest_size:
            raise ValueError('Invalid Ciphertext.')

        if SHA256.new(plain_bytes).digest() != plain_bytes_digest:
            raise ValueError('Invalid Ciphertext.')

        return plain_bytes.decode('utf-8')

if __name__ == '__main__':
    import sys

    def Help():
        print('Usage:')
        print('    SecureCRTCipher.py <enc|dec> [-v2] [-p ConfigPassphrase] <plaintext|ciphertext>')
        print('')
        print('    <enc|dec>              "enc" for encryption, "dec" for decryption.')
        print('                           This parameter must be specified.')
        print('')
        print('    [-v2]                  Encrypt/Decrypt with "Password V2" algorithm.')
        print('                           This parameter is optional.')
        print('')
        print('    [-p ConfigPassphrase]  The config passphrase that SecureCRT uses.')
        print('                           This parameter is optional.')
        print('')
        print('    <plaintext|ciphertext> Plaintext string or ciphertext string.')
        print('                           NOTICE: Ciphertext string must be a hex string.')
        print('                           This parameter must be specified.')
        print('')
    
    def EncryptionRoutine(UseV2 : bool, ConfigPassphrase : str, Plaintext : str):
        try:
            if UseV2:
                print(SecureCRTCryptoV2(ConfigPassphrase).Encrypt(Plaintext))
            else:
                print(SecureCRTCrypto().Encrypt(Plaintext))
            return True
        except:
            print('Error: Failed to encrypt.')
            return False

    def DecryptionRoutine(UseV2 : bool, ConfigPassphrase : str, Ciphertext : str):
        try:
            if UseV2:
                print(SecureCRTCryptoV2(ConfigPassphrase).Decrypt(Ciphertext))
            else:
                print(SecureCRTCrypto().Decrypt(Ciphertext))
            return True
        except:
            print('Error: Failed to decrypt.')
            return False

    def Main(argc : int, argv : list):
        if 3 <= argc and argc <= 6:
            bUseV2 = False
            ConfigPassphrase = ''

            if argv[1].lower() == 'enc':
                bEncrypt = True
            elif argv[1].lower() == 'dec':
                bEncrypt = False
            else:
                Help()
                return -1
            
            i = 2
            while i < argc - 1:
                if argv[i].lower() == '-v2':
                    bUseV2 = True
                    i += 1
                elif argv[i].lower() == '-p' and i + 1 < argc - 1:
                    ConfigPassphrase = argv[i + 1]
                    i += 2
                else:
                    Help()
                    return -1

            if bUseV2 == False and len(ConfigPassphrase) != 0:
                print('Error: ConfigPassphrase is not supported if "-v2" is not specified')
                return -1

            if bEncrypt:
                return 0 if EncryptionRoutine(bUseV2, ConfigPassphrase, argv[-1]) else -1
            else:
                return 0 if DecryptionRoutine(bUseV2, ConfigPassphrase, argv[-1]) else -1
        else:
            Help()

    exit(Main(len(sys.argv), sys.argv))
	
```












