Public details of the Transaction method of AInew Nodes, which is designed to handle and validate transactions coming from external wallets.
The received data structure is as follows:

var transaction = new Transaction
{
    Id = Guid.NewGuid(), // New table identifier
    TransactionId = Guid.NewGuid(), // New unique transaction identifier
    TxHash = "hash", // Currently unused
    SenderAddress = senderWallet.WalletAddress, // Sender wallet address
    RecipientAddress = recipientWallet.Address, // Recipient wallet address
    Amount = Amount, // AInew coin quantity (decimal with 18/2 precision)
    Timestamp = DateTime.UtcNow, // Transaction timestamp
    TransactionType = "send", // Transaction type, in this case "send"
    Status = "in progress", // Transaction status ("in progress" means it is being processed)
    Signature = Signature, // Transaction signature (*1 see below)
    PublicKey = wallet.PublicKey, // Public key needed for signature validation
};

Transaction Sending Code:
using var client = new HttpClient();

// Send the transaction in JSON format
var content = new StringContent(JsonSerializer.Serialize(transaction), Encoding.UTF8, "application/json");

var url = $"http://{activeNode.IPAddress}:{activeNode.Port}/api/Transaction/receive";

// Send the request in JSON format
var response = await client.PostAsync(url, content);

After successful validation, the Node returns a 200 status code! The post-response processing is the responsibility of the wallet's developer.
Signature Validation:
The signature is created during the transaction initiation on the AInew website using the following JavaScript code:

<script>
document.getElementById('signTransaction').addEventListener('click', async function () {
    const amount = document.getElementById('Amount').value;
    const recipient = document.getElementById('RecipientAddress').value;
    const sender = document.getElementById('SenderAddress').value;
    const privateKeyPem = document.getElementById('privateKey').value;

    // Ensure all fields are filled
    if (!amount || !recipient || !sender || !privateKeyPem) {
        alert('Please fill in all required fields.');
        return;
    }

    try {
        // Process PEM format and load private key
        const privateKey = await importPrivateKey(privateKeyPem);

        // Prepare transaction data
        const transactionData = `${amount}-${recipient}-${sender}`;
        const encoder = new TextEncoder();
        const dataBytes = encoder.encode(transactionData);

        // Generate signature
        const signature = await window.crypto.subtle.sign(
            {
                name: "RSASSA-PKCS1-v1_5",
                hash: { name: "SHA-256" }
            },
            privateKey,
            dataBytes
        );

        // Convert signature to Base64 format
        const signatureBase64 = btoa(String.fromCharCode(...new Uint8Array(signature)));
        document.getElementById('Signature').value = signatureBase64;

        // Enable send button
        document.getElementById('sendTransaction').disabled = false;
    } catch (error) {
        console.error("Error signing transaction:", error);
        alert('Error signing transaction: ' + error.message);
        document.getElementById('sendTransaction').disabled = true;
    }
});

// Process and import PEM private key
async function importPrivateKey(pemKey) {
    // Remove PEM header and footer
    const pemHeader = "-----BEGIN PRIVATE KEY-----";
    const pemFooter = "-----END PRIVATE KEY-----";
    const pemBody = pemKey.replace(pemHeader, "").replace(pemFooter, "").replace(/\s+/g, "");
    const binaryDerString = atob(pemBody);
    const binaryDer = new Uint8Array([...binaryDerString].map(char => char.charCodeAt(0)));

    return await window.crypto.subtle.importKey(
        "pkcs8",
        binaryDer.buffer,
        {
            name: "RSASSA-PKCS1-v1_5",
            hash: { name: "SHA-256" }
        },
        true,
        ["sign"]
    );
}
</script>

The JavaScript must be provided with the following inputs:
Quantity of coins.
Sender and recipient wallet addresses.
Private key belonging to the sender's wallet.
Note: External wallet transactions apply only to the quantity of AInew coins. Unique data linked to coins can only be viewed on the AInew website if the external wallet address is registered on the platform.
A procedure is planned to enable the transmission of the unique coin data to external wallets if wallet developers wish to integrate it into their systems.
Remark: If developers use the above JavaScript code to generate the signature, the Node can validate it seamlessly using the corresponding public key.
"The Node API endpoint only accepts requests where every element of the given data structure has a value!
The authentication condition is that the following if statements must be fulfilled:
if (transaction == null)
{
return BadRequest("The transaction is not valid!");
}
if (transaction.Amount - 0.01 <= 0)
{
return BadRequest("The transaction amount must be greater than zero.");
}
if (transaction.Id == null)
{
return BadRequest("The transaction Id must be greater than zero.");
}
if (transaction.TransactionId == null)
{
return BadRequest("The transaction transactionId amount must be greater than zero.");
}
if (transaction.SenderAddress == null)
{
return BadRequest("The transaction SenderAddress must be greater than zero.");
}
if (transaction.RecipientAddress == null)
{
return BadRequest("The transaction RecipientAddress must be greater than zero.");
}
if (transaction.Timestamp == null)
{
return BadRequest("The transaction Timestamp must be greater than zero.");
}
if (transaction.Status == null)
{
return BadRequest("The transaction Status must be greater than zero.");
}
if (transaction.Signature == null)
{
return BadRequest("The transaction Signature must be greater than zero.");
}
if (transaction.PublicKey == null)
{
return BadRequest("The transaction PublicKey must be greater than zero.");
}"
