# 🛡️ SecureShield: TOTP-Powered Multi-Factor Authentication



[![License: CPOL](https://img.shields.io/badge/License-CPOL-blue.svg)](https://opensource.org/licenses/CPOL-1.02)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-July%2015%2C%202024-brightgreen)](https://github.com/nitinprajwal/SecureShield)
[![C#](https://img.shields.io/badge/C%23-239120?style=flat&logo=c-sharp&logoColor=white)](https://docs.microsoft.com/en-us/dotnet/csharp/)

## 📋 Table of Contents

- [Introduction](#-introduction)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Usage](#-usage)
- [How It Works](#-how-it-works)
- [Code Snippets](#-code-snippets)
- [Contributing](#-contributing)
- [License](#-license)
- [Contact](#-contact)

## 🌟 Introduction

SecureShield is a robust Multi-Factor Authentication (MFA) system implementation using the Time-based One-Time Password (TOTP) algorithm as defined in RFC-6238. Designed to integrate seamlessly with popular authenticator apps like Microsoft Authenticator and Google Authenticator, SecureShield provides an additional layer of security for your applications.

## 🚀 Features

- Implementation of RFC-6238 TOTP algorithm
- QR code generation for easy setup with authenticator apps
- Compatibility with Microsoft Authenticator, Google Authenticator, and other TOTP-compliant apps
- PIN + TOTP code combination for enhanced security
- Flexible and easy to integrate into existing C# applications
- Support for both Windows and Linux (via Mono framework)

## 🛠️ Prerequisites

- .NET Framework (version X.X or higher)
- [Net.Codecrete.QrCodeGenerator](https://www.nuget.org/packages/Net.Codecrete.QrCodeGenerator/) NuGet package

## 📥 Installation

1. Clone the repository:
   ```
   git clone https://github.com/nitinprajwal/SecureShield.git
   ```
2. Navigate to the project directory:
   ```
   cd SecureShield
   ```
3. Install required NuGet packages:
   ```
   dotnet restore
   ```

## 🖥️ Usage

1. Generate a TOTP seed and QR code:

```csharp
string seed = getNewId() + getNewId();
seed = seed.Replace("-", "");
seed = seed.Substring(0, 40);

byte[] byteSeed = HexToByte(seed);
var KeyString = Base32.ToBase32String(byteSeed);

// Generate QR code
string tokenURI = string.Format(
    AuthenticatorUriFormat,
    HttpUtility.UrlEncode(orgDomain),
    HttpUtility.UrlEncode(userUPN),
    KeyString);

var qr = QrCode.EncodeText(tokenURI, QrCode.Ecc.High);
```

2. Validate TOTP code:

```csharp
bool isValid = CheckTimeBasedOTP_Rfc6238(byteSeed, incomingOTP);
```

## 🔧 How It Works

1. **Seed Generation**: A unique seed is created using GUIDs for each user.
2. **QR Code Creation**: A QR code is generated containing the seed and user information.
3. **User Registration**: The user scans the QR code with their authenticator app.
4. **TOTP Generation**: The authenticator app generates a new 6-digit code every 30 seconds.
5. **Authentication**: The user enters their PIN + the current TOTP code.
6. **Validation**: The server validates the entered code against the expected TOTP.

## 💻 Code Snippets

### Generate QR Code

```csharp
private void generateQRCode()
{
    string seed = getNewId() + getNewId();
    seed = seed.Replace("-", "");
    seed = seed.Substring(0, 40);

    byte[] byteSeed = HexToByte(seed);

    var KeyString = Base32.ToBase32String(byteSeed);

    string orgDomain = "elogic.synology.me";
    string orgName = "eLogic Builders Inc.";
    string userUPN = "Kashif" + '@' + orgDomain;

    const string AuthenticatorUriFormat = "otpauth://totp/{0}:{1}?secret={2}&issuer={0}&algorithm=SHA1&digits=6&period=30";

    string tokenURI = string.Format(
        AuthenticatorUriFormat,
        HttpUtility.UrlEncode(orgDomain),
        HttpUtility.UrlEncode(userUPN),
        KeyString);

    var qr = QrCode.EncodeText(tokenURI, QrCode.Ecc.High);

    // Convert QR code to base64 encoded SVG
    string base64EncodedImage = Convert.ToBase64String(Encoding.UTF8.GetBytes(qr.ToSvgString(4)));
    string imageSrc = "data:image/svg+xml;base64," + base64EncodedImage;
}
```

### Validate TOTP

```csharp
public bool CheckTimeBasedOTP_Rfc6238(byte[] byteSeed, string incomingOTP)
{
    bool bR = false;
    int IntIncomingCode = int.Parse(incomingOTP);

    var hash = new HMACSHA1(byteSeed);
    var unixTimestamp = Convert.ToInt64(Math.Round((DateTime.UtcNow - new DateTime(1970, 1, 1, 0, 0, 0)).TotalSeconds));
    var timestep = Convert.ToInt64(unixTimestamp / 30);
    
    for (long i = -2; i <= 2; i++)
    {
        var expectedCode = Rfc6238AuthenticationService.ComputeTotp(hash, (ulong)(timestep + i), modifier: null);
        if (expectedCode == IntIncomingCode)
        {
            bR = true;
            break;
        }
    }

    return bR;
}
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the CPOL License. See the [LICENSE](LICENSE) file for details.

## 📞 Contact

Nitin Prajwal - [GitHub](https://github.com/nitinprajwal)
Project Link: [https://github.com/nitinprajwal/SecureShield](https://github.com/nitinprajwal/SecureShield)

---

⭐️ If you find this project useful, please consider giving it a star on GitHub! ⭐️
