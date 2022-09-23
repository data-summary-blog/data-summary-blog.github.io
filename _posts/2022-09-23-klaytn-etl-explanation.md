---
layout: post
title: "Klaytn ETL Intro"
categories: Klaytn
author: "Yongchan Hong"
---

# Klaytn ETL and BigQuery Introduction
## Klaytn ETL
--- 
Klaytn-ETL is the Python scripts for ETL (extract, transform and load) jobs for Klaytn blocks, transactions, ERC20 / ERC721 tokens, transfers, receipts, logs, contracts, and internal transactions. The script is based on Ethereum ETL, and you can check the repository in [Klaytn ETL](https://github.com/klaytn/klaytn-etl). Stars and contributions are welcomed!

## Klaytn ETL Schema (in BigQuery)
--- 

### Blocks  
--- 

|Field|Type|Description|
|------|---|---|
|number|INTEGER|The block number|
|hash|STRING|Hash of the block|
|parent_hash|STRING|Hash of the parent block|
|logs_bloom|STRING|The bloom filter for the logs of the block|
|transaction_root|STRING|The root of the transaction trie of the block|
|state_root|STRING|The root of the final state trie of the block|
|receipts_root|STRING|The root of the receipts trie of the block|
|size|STRING|The size of this block in bytes|
|extra_data|STRING|The extra data field of this block|
|gas_used|NUMERIC|The total used gas by all transactions in this block|
|timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_count|INTEGER|The number of transactions in the block|
|block_score|NUMERIC|Former difficulty. Always 1 in the BFT consensus engine|
|total_block_score|NUMERIC|Integer of the total blockScore of the chain until this block|
|governance_data|STRING|RLP encoded governance configuration|
|vote_data|STRING|RLP encoded governance vote of the proposer|
|committee|ARRAY[STRING]|Array of addresses of committee members of this block. The committee is a subset of validators participated in the consensus protocol for this block|
|proposer|STRING|The address of the block proposer|
|reward_address|STRING|The address of the beneficiary to whom the block rewards were give|
|base_fee_per_gas|NUMERIC|The base fee per gas. This value is returned only when EthTxTypeCompatibleBlock is activated for that block number|

### Transactions
--- 

|Field|Type|Description|
|------|---|---|
|hash|STRING|Hash of the transaction|
|nonce|INTEGER|The number of transactions made by the sender prior to this one|
|block_hash|STRING|Hash of the block|
|block_number|INTEGER|Block number corresponding|
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|from_address|STRING|Address of the sender|
|to_address|STRING|Address of the receiver. null when its a contract creation transaction|
|value|NUMERIC|Value transferred in Wei|
|gas|NUMERIC|Gas provided by the sender|
|gas_price|NUMERIC|Gas price provided by the sender in Wei|
|input|STRING|The data sent along with the transaction|
|fee_payer|STRING|(optional) Address of the fee payer|
|fee_payer_signatures|ARRAY[STRING, STRING, STRING]|(optional) An array of fee payer's signature objects. A signature object contains three fields (V, R, and S). V contains ECDSA recovery id. R contains ECDSA signature r while S contains ECDSA signature s|
|fee_ratio|INTEGER|(optional) Fee ratio of the fee payer. If it is 30, 30% of the fee will be paid by the fee payer. 70% will be paid by the sender|
|sender_tx_hash|STRING|from deserializer|
|signatures|ARRAY[STRING, STRING, STRING]|An array of signature objects. A signature object contains three fields (V, R, and S). V contains ECDSA recovery id. R contains ECDSA signature r while S contains ECDSA signature s|
|tx_type|STRING|A string representing the type of the transaction|
|tx_type_int|INTEGER|An integer representing the type of the transaction|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|receipt_gas_used|NUMERIC|The amount of gas used by this specific transaction alone|
|receipt_contract_address|STRING|The contract address created, if the transaction was a contract creation, otherwise null|
|receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|dw_load_dt|TIMESTAMP|The UTC time when the data is inserted|
|max_priority_fee_per_gas|NUMERIC|A maximum amount to pay for the transaction to execute|
|max_fee_per_gas|NUMERIC|Gas tip cap for dynamic fee transaction in peb|
|access_list| ARRAY[STRING, ARRAY[STRING]] |An array of accessList|

### Logs
---

|Field|Type|Description|
|------|---|---|
|block_number|INTEGER|Block number corresponding|
|block_hash|STRING|Hash of the block|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|transaction_hash|STRING|Hash of the transactions|
|transaction_receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|log_index|INTEGER|Integer of the log index position in the block|
|address|STRING|Address from which this log originated|
|data|STRING|Contains one or more 32 Bytes non-indexed arguments of the log|
|topics|ARRAY[STRING]|Indexed log arguments (0 to 4 32-byte hex strings). (In solidity: The first topic is the hash of the signature of the event (e.g. Deposit(address,bytes32,uint256)), except you declared the event with the anonymous specifier.)|

### Token Transfers
---

|Field|Type|Description|
|------|---|---|
|token_address|STRING|Token address|
|from_address|STRING|Address of the sender|
|to_address|STRING|Address of the receiver|
|value|DECIMAL|Amount of tokens transferred (ERC20) / id of the token transferred (ERC721). Use safe_cast for casting to NUMERIC or FLOAT64|
|block_hash|STRING|Hash of the block|
|block_number|INTEGER|Block number corresponding|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_hash|STRING|Hash of the transactions|
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|transaction_receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|log_index|INTEGER|Integer of the log index position in the block|

### Tokens

|Field|Type|Description|
|------|---|---|
|address|STRING|The address of the token|
|symbol|STRING|The symbol of the token|
|name|STRING|The name of the token|
|decimals|INTEGER|The number of decimals the token uses. Use safe_cast for casting to NUMERIC or FLOAT64|
|total_supply|NUMERIC|The total token supply. Use safe_cast for casting to NUMERIC or FLOAT64|
|function_sighashes|ARRAY[STRING]|4-byte function signature hashes|
|is_erc20|BOOLEAN|Whether this contract is an ERC20 contract|
|is_erc721|BOOLEAN|Whether this contract is an ERC721 contract|
|block_number|INTEGER|Block number corresponding|
|block_hash|STRING|Hash of the block|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_hash|STRING|Hash of the transactions |
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|transaction_receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|trace_index|INTEGER|Trace Index of the trace|
|trace_status|INTEGER|Either 1 (success) or 0 (failure, due to any operation that can cause the call itself or any top-level call to revert)|
|creator_address|STRING|Token creator address|

### Contracts

|Field|Type|Description|
|------|---|---|
|address|STRING|Address of the contract|
|bytecode|STRING|Bytecode of the contract|
|function_sighashes|ARRAY[STRING]|4-byte function signature hashes|
|is_erc20|BOOLEAN|Whether this contract is an ERC20 contract|
|is_erc721|BOOLEAN|Whether this contract is an ERC721 contract|
|block_number|INTEGER|Block number corresponding|
|block_hash|STRING|Hash of the block|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_hash|STRING|Hash of the transactions |
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|transaction_receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|trace_index|INTEGER|Trace Index of the trace|
|trace_status|INTEGER|Either 1 (success) or 0 (failure, due to any operation that can cause the call itself or any top-level call to revert)|
|creator_address|STRING|Token creator address|

### Traces
---

|Field|Type|Description|
|------|---|---|
|block_number|INTEGER|Block number corresponding|
|block_hash|STRING|Hash of the block|
|block_timestamp|TIMESTAMP|The UTC timestamp for when the block was collated|
|block_unix_timestamp|FLOAT|The unix timestamp for when the block was collated|
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|transaction_hash|STRING|Hash of the transactions|
|transaction_receipt_status|INTEGER|Either 1 (success) or 0 (failure) (post Byzantium)|
|from_address|STRING|Address of the sender, null when trace_type is reward|
|to_address|STRING|Address of the receiver if trace_type is call, address of new contract or null if trace_type is create, beneficiary address if trace_type is suicide, miner address if trace_type is reward|
|value|NUMERIC|Value transferred in Wei|
|input|STRING|The data sent along with the message call|
|output|STRING|The output of the message call, bytecode of contract when trace_type is create|
|trace_type|STRING|One of call, create, suicide, reward|
|call_type|STRING|One of call, callcode, delegatecall, staticcall|
|gas|NUMERIC|Gas provided with the message call|
|gas_used|NUMERIC|Gas used by the message call|
|subraces|INTEGER|Number of subtraces|
|trace_address|ARRAY[STRING]|Comma separated list of trace address in call tree|
|error|STRING|Error message|
|status|INTEGER|Either 1 (success) or 0 (failure, due to any operation that can cause the call itself or any top-level call to revert)|
|trace_index|INTEGER|Trace index of the trace|

### Receipts
---

|Field|Type|Description|
|------|---|---|
|transaction_hash|STRING|Hash of the transactions|
|transaction_index|INTEGER|Integer of the transactions index position in the block|
|block_hash|STRING|Hash of the block|
|block_number|INTEGER|Block number corresponding|
|gas|NUMERIC|Gas provided by the sender|
|gas_price|NUMERIC|Gas price provided by the sender in Wei|
|gas_used|NUMERIC|The total used gas by all transactions in this block|
|effective_gas_price|NUMERIC|The actual value per gas deducted from the senders account|
|contract_address|STRING|The contract address created, if the transaction was a contract creation, otherwise null|
|logs_bloom|STRING|The bloom filter for the logs|
|nonce|INTEGER|The number of transactions made by the sender prior to this one|
|fee_payer|STRING|(optional) Address of the fee payer|
|fee_payer_signatures|ARRAY[STRING, STRING, STRING]|(optional) An array of fee payer's signature objects. A signature object contains three fields (V, R, and S). V contains ECDSA recovery id. R contains ECDSA signature r while S contains ECDSA signature s|
|fee_ratio|INTEGER|(optional) Fee ratio of the fee payer. If it is 30, 30% of the fee will be paid by the fee payer. 70% will be paid by the sender|
|code_format|STRING|(optional) The code format of smart contract code|
|human_readable|BOOLEAN|(optional) true if the address is humanReadable, false if the address is not humanReadable|
|tx_error|STRING|(optional) detailed error code if status is equal to zero|
|key|STRING|(optional) Key of the newly created account|
|input|STRING|(optional) The data sent along with the transaction|
|from|STRING|Address of the sender|
|to|STRING|Address of the receiver. Null when it is a contract creation transaction|
|type_name|STRING|A string representing the type of the transaction|
|type_int|INTEGER|An integer representing the type of the transaction|
|sender_tx_hash|STRING|Hash of a transaction that is signed only by the sender|
|fee_payer_signatures|ARRAY[STRING, STRING, STRING]|An array of signature objects. A signature object contains three fields (V, R, and S). V contains ECDSA recovery id. R contains ECDSA signature r while S contains ECDSA signature s|
|status|INTEGER|Either 1 (success) or 0 (failure)|
|value|NUMERIC|(optional) Integer of the value sent with this transaction|
|input_json|ARRAY|(optional) The data sent along with the transaction|
|access_list| ARRAY[STRING, ARRAY[STRING]] |An array of accessList|
|chain_id|INTEGER|id of the chain|
|max_priority_fee_per_gas|NUMERIC|A maximum amount to pay for the transaction to execute|
|max_fee_per_gas|NUMERIC|Gas tip cap for dynamic fee transaction in peb|

## Klaytn ETL in Airflow
---
Klaytn runs `enrich_block_group` and `enrich_trace_group` CLIs in hourly batch using Airflow. The data will be exported in JSON.GZ format in GCS. From GCS, Klaytn create s external table in BigQuery.

## Klaytn BigQuery Dataset
---
Currently 8 tables are available, containing data from 2022-09-14 to now.
- data-test-361602.klaytn_data_test.cypress_blocks_table
- data-test-361602.klaytn_data_test.cypress_transactions_table
- data-test-361602.klaytn_data_test.cypress_traces_table
- data-test-361602.klaytn_data_test.cypress_tokens_table
- data-test-361602.klaytn_data_test.cypress_token_transfers_table
- data-test-361602.klaytn_data_test.cypress_contracts_table
- data-test-361602.klaytn_data_test.cypress_receipts_table
- data-test-361602.klaytn_data_test.cypress_logs_table

You can query like following 
```
### Daily Active Address Count
WITH TOTAL_ADDRESS AS (
SELECT 
  CAST(block_timestamp as DATE) as block_date,
  from_address as address
FROM
  data-test-361602.klaytn_data_test.cypress_transactions_table
WHERE 
  from_address IS NOT NULL
UNION ALL
select
    CAST(block_timestamp as DATE) as block_date,
    to_address as address
FROM
  data-test-361602.klaytn_data_test.cypress_transactions_table
WHERE 
  from_address IS NOT NULL)
SELECT 
  block_date, 
  COUNT(address) as active_address_count
FROM 
  TOTAL_ADDRESS
GROUP BY
  block_date
ORDER BY
  block_date
```
Since dataset is not optimized for now, query may take some time (~30s)

## TODO
- Support streaming
- Make query faster with optimization
- Add block date, block hour