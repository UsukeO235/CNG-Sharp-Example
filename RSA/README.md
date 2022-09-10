# CNG APIでRSA鍵を扱うサンプル
## 秘密鍵を生成・永続化し、文字列を暗号化する
```c#
using System.Security.Cryptography;

CngProvider provider = new CngProvider("Microsoft Software Key Storage Provider");

CngKeyCreationParameters parameters = new CngKeyCreationParameters();
parameters.Provider = provider;
parameters.ExportPolicy = CngExportPolicies.None;
parameters.KeyCreationOptions = CngKeyCreationOptions.None;
parameters.KeyUsage = CngKeyUsages.Decryption;

// CngKey.Createの前に鍵長を設定する
CngProperty property =  new CngProperty("Length", BitConverter.GetBytes(3072), CngPropertyOptions.None);
parameters.Parameters.Add(property);

CngKey private_key;

using(private_key = CngKey.Create(CngAlgorithm.Rsa, "test-20220903", parameters))
{
    RSACng rsa_cng = new RSACng(private_key);
    
    String original = "ABCDEFG";  // 平文
    byte[] encrypted = rsa_cng.Encrypt(System.Text.Encoding.ASCII.GetBytes(original), RSAEncryptionPadding.OaepSHA256);  // 暗号化
}
```

## 永続化した秘密鍵から公開鍵を取得し、暗号文を復号化する
```c#
using System.Security.Cryptography;

CngProvider provider = new CngProvider("Microsoft Software Key Storage Provider");

CngKeyCreationParameters parameters = new CngKeyCreationParameters();
parameters.Provider = provider;
parameters.ExportPolicy = CngExportPolicies.None;
parameters.KeyCreationOptions = CngKeyCreationOptions.None;
parameters.KeyUsage = CngKeyUsages.Decryption;

CngKey private_key;

if(CngKey.Exists("test-20220903", provider))
{
    using(private_key = CngKey.Open("test-20220903", provider))
    {
        RSACng rsa_cng = new RSACng(private_key);
        
        // 公開鍵取得
        byte[] public_key_blob = private_key.Export(CngKeyBlobFormat.GenericPublicBlob);
        CngKey public_key = CngKey.Import(public_key_blob, CngKeyBlobFormat.GenericPublicBlob, provider);

        RSACng rsa_cng = new RSACng(public_key);
        
        byte[] encrypted;  // 暗号文(RSACng(private_key).Encrypt()で得た暗号文)
        byte[] decrypted = rsa_cng.Decrypt(encrypted, RSAEncryptionPadding.OaepSHA256);  // 復号化
    }
}
```
