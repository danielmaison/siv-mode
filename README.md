# Java RFC 5297 SIV Authenticated Encryption

[![Build](https://github.com/cryptomator/siv-mode/workflows/Build/badge.svg)](https://github.com/cryptomator/siv-mode/actions?query=workflow%3ABuild)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=cryptomator_siv-mode&metric=alert_status)](https://sonarcloud.io/dashboard?id=cryptomator_siv-mode)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=cryptomator_siv-mode&metric=coverage)](https://sonarcloud.io/dashboard?id=cryptomator_siv-mode)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=cryptomator_siv-mode&metric=vulnerabilities)](https://sonarcloud.io/dashboard?id=cryptomator_siv-mode)
[![Maven Central](https://img.shields.io/maven-central/v/org.cryptomator/siv-mode.svg?maxAge=86400)](https://repo1.maven.org/maven2/org/cryptomator/siv-mode/)
[![Javadocs](http://www.javadoc.io/badge/org.cryptomator/siv-mode.svg)](http://www.javadoc.io/doc/org.cryptomator/siv-mode)

## Features
- No dependencies (required BouncyCastle classes are repackaged)
- Passes official RFC 5297 test vectors
- Constant time authentication
- Defaults on AES, but supports any block cipher with a 128-bit block size.
- Supports any key sizes that the block cipher supports (e.g. 128/192/256-bit keys for AES)
- Thread-safe
- [Fast](https://github.com/cryptomator/siv-mode/issues/15)
- Requires JDK 8+ or Android API Level 24+ (since version 1.4.0)

## Audits
- [Version 1.0.8 audit by Tim McLean](https://www.chosenplaintext.ca/publications/20161104-siv-mode-report.pdf) (Issues fixed with 1.1.0)
- [Version 1.2.1 audit by Cure53](https://cryptomator.org/audits/2017-11-27%20crypto%20cure53.pdf)

| Finding | Comment |
|---|---|
| 1u1-22-001 | The GPG key is used exclusively for the Maven repositories, is designed for signing only and is protected by a 30-character generated password (alphabet size: 96 chars). It is iterated and salted (SHA1 with 20971520 iterations). An offline attack is also very unattractive. Apart from that, this finding has no influence on the Tresor apps<sup>[1](#footnote-tresor-apps)</sup>. This was not known to Cure53 at the time of reporting. |
| 1u1-22-002 | As per contract of `BlockCipher#processBlock(byte[], int, byte[], int)`, `JceAesBlockCipher` is designed to encrypt or decrypt just **one single block** at a time. JCE doesn't allow us to retrieve the plain cipher without a mode, so we explicitly request `AES/ECB/NoPadding`. This is by design, because we want the plain cipher for a single 128 bit block without any mode. We're not actually using ECB mode. |

## Usage
```java
private static final SivMode AES_SIV = new SivMode();

public void encrypt() {
  byte[] encrypted = AES_SIV.encrypt(ctrKey, macKey, "hello world".getBytes());
  byte[] decrypted = AES_SIV.decrypt(ctrKey, macKey, encrypted);
}

public void encryptWithAssociatedData() {
  byte[] encrypted = AES_SIV.encrypt(ctrKey, macKey, "hello world".getBytes(), "associated".getBytes(), "data".getBytes());
  byte[] decrypted = AES_SIV.decrypt(ctrKey, macKey, encrypted, "associated".getBytes(), "data".getBytes());
}
```

## Maven integration

```xml
<dependencies>
  <dependency>
    <groupId>org.cryptomator</groupId>
    <artifactId>siv-mode</artifactId>
    <version>1.4.0</version>
  </dependency>
</dependencies>
```

## JPMS

From version 1.3.2 onwards this library is an explicit module with the name `org.cryptomator.siv`. You can use it by adding the following line to your `module-info.java`.

```java
requires org.cryptomator.siv;
```

Because BouncyCastle classes are shaded, this library only depends on `java.base`.

## Building

This is a Maven project. To build it, run `mvn clean install`.

Requires JDK 11.0.3 or newer at build time due to JPMS support.

## License
Distributed under the MIT X Consortium license. See the LICENSE file for more info.

---

<sup><a name="footnote-tresor-apps">1</a></sup> The Cure53 pentesting was performed during the development of the apps for 1&1 Mail & Media GmbH.
