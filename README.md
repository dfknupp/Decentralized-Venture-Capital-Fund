# Decentralized-Venture-Capital-Fund
構築WEB3ベースのベンチャーキャピタルファンドにより、個人投資家がスタートアップ企業に直接投資することができます。
from web3 import Web3
import json

# Connect to local Ethereum node (Ganache, in this case)
ganache_url = "HTTP://127.0.0.1:7545"
web3 = Web3(Web3.HTTPProvider(ganache_url))

# Check if connected to blockchain
if web3.isConnected():
    print("Connected to blockchain.")
else:
    print("Failed to connect to blockchain.")

# Set up the contract
contract_address = web3.toChecksumAddress("YOUR_CONTRACT_ADDRESS_HERE")
with open("VentureCapitalFundABI.json", "r") as file:
    contract_abi = json.load(file)

# Creating a contract object
vc_fund_contract = web3.eth.contract(address=contract_address, abi=contract_abi)

# Function to allow an investor to invest in a startup
def invest_in_startup(investor_private_key, startup_id, investment_amount_eth):
    # Convert investment amount from ETH to Wei
    investment_amount_wei = web3.toWei(investment_amount_eth, "ether")

    # Get investor's account from private key
    investor_account = web3.eth.account.privateKeyToAccount(investor_private_key)

    # Build transaction
    txn_dict = vc_fund_contract.functions.invest(startup_id).buildTransaction({
        'chainId': 1337,  # Chain ID for Ganache
        'gas': 2000000,
        'gasPrice': web3.toWei('50', 'gwei'),
        'nonce': web3.eth.getTransactionCount(investor_account.address),
        'value': investment_amount_wei
    })

    # Sign the transaction
    signed_txn = investor_account.signTransaction(txn_dict)

    # Send the transaction
    txn_receipt = web3.eth.sendRawTransaction(signed_txn.rawTransaction)

    # Wait for the transaction to be mined
    txn_receipt = web3.eth.waitForTransactionReceipt(txn_receipt)

    if txn_receipt['status'] == 1:
        print("Investment successful.")
    else:
        print("Investment failed.")

# Example usage (Replace values with actual ones)
investor_private_key = "YOUR_INVESTOR_PRIVATE_KEY_HERE"
startup_id = 1  # Assuming IDs are used to identify startups
investment_amount_eth = 0.5  # Amount in ETH

# Call the function to invest in a startup
invest_in_startup(investor_private_key, startup_id, investment_amount_eth)
