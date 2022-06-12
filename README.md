# **Guideline**

**Purpose:** To create multiple tokens and offer files with fees on the Chia Blockchain

Since every coin must have one `unique coin id` derived from `parent_coin_id`, `puzzle_hash`, and `amount`, we can't create multiple small coins with the same information. We will have to acquire different `puzzle hash` via `chia keys derive`.  This will allow us to create one transaction that will spend one big coin and create multiple small coins.  We will then push these small coins to the blockchain and to our wallet where we host our XCH fees to create the offer files.   

The set of scripts will go through the process of:
- Preparing the XCH fees for individual CATS that are to be used for the offer files
- Breaking the CATS into individual small tokens
- Creating offer files for each individual tokens to host on a decentralized exchange platform  

---
### This guide walks through the steps of prepaing 200 CATs and creating 200 offer files with .0005 fees each. 
---

**Environment:**
- Linux, Ubuntu 20.04 LTS

**You will need:**
- Powershell: https://docs.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.2
- A sync Chia Node
- 2 synced Chia wallets
- XCH for fees
- 200 CAT
- chia-dev-tools to run `cdv` or `chia rpc` commands: https://github.com/Chia-Network/chia-dev-tools 
- Scripts: 
  - (Script #1) XCH Breaking for Fees: https://gist.github.com/kimsk/c78cab395fe0a5369f380ed97194dcc8
  - (Script #2) Brute Force Breaking CATS: https://gist.github.com/kimsk/5e5e32210eab7390ab15ada02fa798b5 
  - (Script #3) Issuing the offer files with fees: https://gist.github.com/kimsk/0a2604b2a5999fa092b1bb50b77e0624 

# **Steps**
## **Script 1: Breaking Fees**

### **Generate a new key** 

Get into your chia-blockchain folder and activate the command lines.  You will need to create a new wallet key to host the fees.  Note down the wallet information and we will need this for the remaining of this guidleine.

```
$ cd chia-blockchain
$ . ./activate
$ chia keys genereate

# To show your wallet details
$ chia keys show

# To show your mnemonic seed
$ chia keys show --show-mnemonic-seed
```

### **Send 1 XCH to a new wallet to prepare the fees for offer files** 

To improve the transaction time, its best to use .00005 XCH for the transaction fee.

You can get all of your wallet's information using `chia wallet show` and `$chia wallet show -h` for more information.

In put your information for the following inputs
```
-f, --fingerprint INTEGER       Set the fingerprint to specify which wallet to use

-i, --id INTEGER                Id of the wallet to use 

-a, --amount TEXT               How much chia to send, in XCH        

-t, --address TEXT              Address to send the XCH

-m, --fee TEXT                  Set the fees for the transaction, in XCH
```

```
$ chia wallet send -f <input> -i <input> -a <input> -m .0005 -t <input>
```

### **Confirm the transaction**

You will be presented with the option to check the status of transaction if its `Confirmed`.

```
$ chia wallet get_transaction -f <input> -tx <insert the tx> 

## Example of output:

Transaction 11663dd6dbd689c3297db5ce8a43613c50a30c276705a8bc3d4eb1e852e86fe5
Status: Confirmed
Amount sent: 1 XCH
To address: txch14nz35200d2eet8pl4eds0dp3l2esvyx2cqjsrdhup4l3tj0x55yqm58j2f
Created at: 2022-06-04 13:12:04
```

### **Activate python3 to run chia-dev tools command**

If you don't have chia-dev tools, you can get it from here: https://github.com/Chia-Network/chia-dev-tools.
After installing chia-dev tools, you will need to activate python to run the cmds:  

```
$ python3 -m venv venv
$ . ./venv/bin/activate
$ pip install chia-dev-tools
```

### **Check the coin's puzzel hash** 

So, we just created a new wallet and sent 1 XCH to it. The 1 XCH coin is encoded with a puzzle hash. You can see the puzzle has with the `cdv decode` and `cdv rpc coinrecords` command;

```
❯ cdv decode xch14nz35200d2eet8pl4eds0dp3l2esvyx2cqjsrdhup4l3tj0x55yqm58j2f
acc51a29ef6ab3959c3fae5b07b431fab30610cac02501b6fc0d7f15c9e6a508
```

```
$ cdv rpc coinrecords --by puzzlehash $(cdv decode xch14nz35200d2eet8pl4eds0dp3l2esvyx2cqjsrdhup4l3tj0x55yqm58j2f)
```

### **Check your wallet balance for the 1 XCH we sent**

```
$ chia wallet show
``` 

You should see your `1 XCH`.  

This `1 XCH` represents `1 big coin`. When you create an offer, the offer will lock this coin. 

Even if you only send `1 mojo`, the `1 XCH` will be locked by your wallet.  This is how the `coin set model` is different from the `account model` thats used by Etherium Blockchain.

### **Break XCH into small individual coins** 

We will need to create a ps1 file. We've created the file here on your `/home`

```
$ cd ~
$ sudo nano xch_breaking.ps1
```

Copy and paste the code from this git into the file:
https://gist.github.com/kimsk/c78cab395fe0a5369f380ed97194dcc8

You will need to input your information in the following section:
- $FINGERPRINT: input the fingerprint of your wallet that holds the 1 XCH
- $NUM: input the number of transaction you plan to create, in our case its 190 (since we want to create 190 offer files for our cats) 


```
$WALLET_ID = 1
$NUM = 190
$AMOUNT = $FEE
```
- save and exit the file (Press CTRL + O, Enter, CTRL + X, Enter)

You will need to run this script using `powershell`.  If you don't have it, you can download it here: https://docs.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.2  


Enter powershell and run the script.

```
$ pwsh
```

Keep in mind, if you saved the script in a location other than home, you'll need to adjust this line according to the location of the file.

```
$ ./xch_breaking.ps1
```

At this point, you will see the `spend_bundle` which is sent to the blockchain and processed by every full node.To see that your big coin was spent, you can use the `cdv rpc conrecords` command and your wallet address. 

```
$ cdv rpc coinrecords --by puzzlehash $(cdv decode <input wallet address>)
```

You can also check all the small coins address using `chia keys derive` and your wallet's `fingerprint`

```
$ chia keys derive -f <fingerprint> wallet-address -n 10
```

Pick one of the address and run `cdv rpc coinrecords` command 

```
$ cdv rpc coinrecords --by puzzlehash $(cdv decode <input wallet address>)
```


## **Script 2: Breakding CATS**
First lets verify that we have one large 200 CAT coins using `cdv rp coinrecords` commpand 

You will need the `CAT ID`  and the `WALLET ADDRESS` that host this `CAT` to check this. 

``` 
cdv rpc coinrecords --by puzzlehash $(cdv clsp cat_puzzle_hash --tail <input TOKEN ID> $(cdv decode <input wallet address>))
```

You should get an output that shows the details of your `CAT` 

```
[
    {
        "coin": {
            "amount": 200000,
            "parent_coin_info": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "puzzle_hash": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        },
        "coinbase": false,
        "confirmed_block_index": ###########,
        "spent_block_index": 0,
        "timestamp": ###########
    }
]
```
So, now we can prepare the script and run it.

Create a file named `brute-force-cats-breaking-phase1.ps1` on your `home` then copy and paste the following code which you can get from this git: https://gist.github.com/kimsk/5e5e32210eab7390ab15ada02fa798b5 . 

Keep in mind the origional file consist of codes with `Phase 1` & `Phase 2`, in this step we will only be using `Phase 1`.

Open the file we just created and copy the code in.

```
$ sudo nano brute-force-cats-breaking.ps1
```

**Phase 1**
```
$sw = new-object system.diagnostics.stopwatch
$sw.Start()

$FINGERPRINT = <input your info>
$WALLET_ID = <input your info>
$WALLET_ID_JSON = (@{ wallet_id = $WALLET_ID } | ConvertTo-Json)  -replace '"', '\""'
$FEE = <input your info> # for example 50_000_000 mojos is 0.00005 XCH
$FEE_WALLET_ID_JSON = (@{ wallet_id = 1 } | ConvertTo-Json)  -replace '"', '\""'
$AMOUNT = 1000 # one CAT is 1000 mojos

# set synced wallet
chia wallet show -f $FINGERPRINT | Out-Null
Start-Sleep -s 5

$NUM = <input your info>
$bin_count = [Math]::Ceiling([Math]::Log($NUM) / [Math]::Log(2))

# Phase One
$sw_1 = new-object system.diagnostics.stopwatch
$sw_1.Start()
# 2^n coins
for ($i=1;$i -lt $bin_count;$i++){
    $count = [Math]::Pow(2, $i)
    $amt = [int](($NUM/$count) * $AMOUNT)
    $start = $count - 1
    Write-Host "========="
    Write-Host "$i, $start : $count coins"
    Write-Host "========="
    $addresses =
        chia keys derive -f $FINGERPRINT wallet-address -i $start -n $count
        | % { $_ -replace '(^Wallet address )(.*)(: )', "" }
    
    for($j=0;$j -lt $count;$j++) {
        $idx = $start + $j
        Write-Host "$idx : $amt to $($addresses[$j])" 
    
        $not_enough_spendable = $True
        while($not_enough_spendable){
            $spendable_amount = (chia rpc wallet get_wallet_balance $WALLET_ID_JSON | ConvertFrom-Json).wallet_balance.spendable_balance
            $not_enough_amount =  $spendable_amount -lt $AMOUNT
            $spendable_fee = (chia rpc wallet get_wallet_balance $FEE_WALLET_ID_JSON | ConvertFrom-Json).wallet_balance.spendable_balance
            $not_enough_fee =  $spendable_fee -lt $FEE
            if($not_enough_amount -or $not_enough_fee){
                Start-Sleep -s 5
                Write-Host "spendable_amount: $spendable_amount spendable_fee: $spendable_fee"
            }
            else {
                $not_enough_spendable = $False
            }
        }

        $address = $addresses[$j]
        $cat_spend_json = (@{
            fingerprint = $FINGERPRINT
            wallet_id = $WALLET_ID
            amount = $amt
            fee = $FEE
            inner_address = $address
        } | ConvertTo-Json) -replace '"', '\""'
        # $cat_spend_json
        $result = chia rpc wallet cat_spend $cat_spend_json | ConvertFrom-Json
        Write-Host "txn_id: $($result.transaction_id)"
    }
}
$sw_1.Stop()
```

You will need to input your information for `FINGERPRINT`, `WALLET_ID`, `FEE`, `NUM`:

- $FINGERPRINT = fingerprint of the wallet that hosts your CAT.  You can check using `chia wallet show` 
- $WALLET_ID = You can check the wallet ID of your CAT using `chia wallet show` command 
- $FEE = This will be the fee for breaking the CAT and pushing it to the blockchain, we used `.0005 XCH` here. 
- $NUM = The number of CAT you want to break into, in our case we used `200`

now access `powershell` and run the script

```
$ pwsh
$ ./brute-force-cats-breaking.ps1
```

Once you noticed the output shows the number of CAT you wanted, (In our case its `200`)  you can stop it with `CTRL + C` and exit out of `powershell`

Now you will need remove the code from `Phase 1` code and input `phase 2` codes to run it in `powershell` again.  


Open up your brute-force-cats-breaking.ps1 file.

```
$ sudo nano brute-force-cats-breaking.ps1
```

Delete everything in `brute-force-cats-breaking-phase1.ps1` except for 

```
$FINGERPRINT = <input your info>
$WALLET_ID = <input your info>
$WALLET_ID_JSON = (@{ wallet_id = $WALLET_ID } | ConvertTo-Json)  -replace '"', '\""'
$FEE = <input your info> # for example 50_000_000 mojos is 0.00005 XCH
$FEE_WALLET_ID_JSON = (@{ wallet_id = 1 } | ConvertTo-Json)  -replace '"', '\""'
$AMOUNT = 1000 # one CAT is 1000 mojos
```

Copy in this below the codes above and make sure you input the `wallet address` that hosts your XCH fees from `Script 1: Breaking XCH Fees`:

```
# Phase Two
$sw_2 = new-object system.diagnostics.stopwatch
$sw_2.Start()
$FROM_FINGERPRINT = $FINGERPRINT # CAT wallet
$TO_ADDRESS = "<input wallet address>" # wallet to create offers, same wallet where you stored your XCH fees from script 1
$FROM_WALLET_ID = $WALLET_ID

# set synced wallet
chia wallet show -f $FROM_FINGERPRINT | Out-Null
Start-Sleep -s 5

# break coin
for(($i=0);$i -lt $NUM;$i++)
{
    $not_enough_spendable = $True
    while($not_enough_spendable){
        $spendable_amount = (chia rpc wallet get_wallet_balance $WALLET_ID_JSON | ConvertFrom-Json).wallet_balance.spendable_balance
        $enough_amount =  $spendable_amount -ge $AMOUNT
        $spendable_fee = (chia rpc wallet get_wallet_balance $FEE_WALLET_ID_JSON | ConvertFrom-Json).wallet_balance.spendable_balance
        $enough_fee =  $spendable_fee -ge $FEE
        if($enough_amount -and $enough_fee){
            $not_enough_spendable = $False
        }
        Start-Sleep -s 5
        Write-Host "spendable_amount: $spendable_amount spendable_fee: $spendable_fee"
    }

    $cat_spend_json = (@{ 
        fingerprint = $FROM_FINGERPRINT
        wallet_id = $FROM_WALLET_ID
        amount = $AMOUNT
        fee = $FEE
        inner_address = $TO_ADDRESS
    } | ConvertTo-Json) -replace '"', '\""'
    # Write-Host $cat_spend_json
    $result = chia rpc wallet cat_spend $cat_spend_json | ConvertFrom-Json
    Write-Host "txn_id: $($result.transaction_id)"
}


$sw_2.Stop()
$sw.Stop()
Write-Host $sw_1.Elapsed.TotalMinutes
Write-Host $sw_2.Elapsed.TotalMinutes
Write-Host $sw.Elapsed.TotalMinutes
```

## **Script 3: Issuing the offer fies for your CAT**
create a file and name it offers_creating.ps1

```
$ sudo nano offers_creating.ps1
```

Copy the codes from https://gist.github.com/kimsk/0a2604b2a5999fa092b1bb50b77e0624 and paste them into the files

edit the `NUM`, `FINGERPRINT`, and `WALLET ID` `AMOUNT` according to where your CATS & XCH Fees are located in.  Also edit `$OFFER_PATH =` to where you want the file to be saved to and `$offer_file_name`for the name of the file. Like so below; 

```
$OFFER_FINGERPRINT = 55544433
$OFFER_WALLET_ID = 2
$FEE = 500000000
$NUM = 190
$OFFER_PATH = "~"
$offer_file_name = "1TEST_x_1XCH-$i.offer"
```

## **Cancelling all offer files**

If anything goes wrong when creating the offer files, you can cancel then all and start over using tgis script.

https://github.com/kimsk/chia-concepts/blob/main/misc/prepare-offers/cancel_all_offers.ps1

Create a file called cancel_all_offers.ps1
 
```
$sudo nano cancel_all_offers.ps1
```

run the script in powershell
