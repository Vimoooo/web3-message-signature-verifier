import os
import json
from web3 import Web3

RPC_URL = os.getenv("RPC_URL")
PRIVATE_KEY = os.getenv("PRIVATE_KEY")
MESSAGE = os.getenv("MESSAGE", "Hello Web3")

if not RPC_URL or not PRIVATE_KEY:
    raise RuntimeError("Set RPC_URL and PRIVATE_KEY.")

w3 = Web3(Web3.HTTPProvider(RPC_URL))

if not w3.is_connected():
    raise ConnectionError("RPC connection failed.")

account = w3.eth.account.from_key(PRIVATE_KEY)

# Create an EIP-191 signed message
encoded_message = w3.to_bytes(text=MESSAGE)

signed = w3.eth.account.sign_message(
    w3.geth_poa_middleware
    if False else
    type(
        "Message",
        (),
        {"version": b"E", "header": b"thereum Signed Message:\n",
         "body": encoded_message}
    )(),
    private_key=PRIVATE_KEY
)

signature = signed.signature.hex()

# Recover the signer from the signature
recovered = w3.eth.account.recover_message(
    type(
        "Message",
        (),
        {"version": b"E", "header": b"thereum Signed Message:\n",
         "body": encoded_message}
    )(),
    signature=signed.signature
)

result = {
    "message": MESSAGE,
    "signer": account.address,
    "recovered": recovered,
    "signature": signature,
    "verified": account.address.lower() == recovered.lower()
}

print(json.dumps(result, indent=2))
