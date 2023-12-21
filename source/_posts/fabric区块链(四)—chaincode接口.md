---
title: fabric区块链(四)—chaincode接口
date: 2023/5/13
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/22.jpg
---

# fabric智能合约-chaincode接口：

## 

```go
package shim

import (
	"github.com/hyperledger/fabric/protos/peer"
)

// ChaincodeStubInterface provides the shim interface of a chaincode invocation
type ChaincodeStubInterface interface {
	// State Management

	// GetState returns the value of the specified `key`
	GetState(key string) ([]byte, error)
	// PutState writes the specified `value` and `key` into the ledger
	PutState(key string, value []byte) error
	// DelState deletes the specified `key` from the ledger
	DelState(key string) error

	// Private Data Management

	// GetPrivateData returns the value of the specified `key` from the private data collection
	GetPrivateData(collection, key string) ([]byte, error)
	// PutPrivateData writes the specified `value` and `key` into the private data collection
	PutPrivateData(collection, key string, value []byte) error
	// DelPrivateData deletes the specified `key` from the private data collection
	DelPrivateData(collection, key string) error

	// State Query

	// GetStateByRange returns a range iterator over a set of keys in the ledger. The iterator can be used to iterate over all keys
	// between the startKey (inclusive) and endKey (exclusive). However, if the number of keys between startKey and endKey is greater than
	// the totalQueryLimit then iterator will be positioned at startKey and will contain no keys. The totalQueryLimit is intended to be
	// set by a chaincode's MSP policies at install or instantiation time. If no value is set by the policy, then the default value of
	// the peer will be used. Setting totalQueryLimit to zero will allow a chaincode to read any number of keys.
	GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)
	// GetStateByPartialCompositeKey queries the state in the ledger based on a given partial composite key. This function returns an
	// iterator which can be used to iterate over all composite keys whose prefix matches the given partial composite key. The
	// `ObjectType` returned by the iterator is the `Attributes`. If there are no matching keys in the ledger, then the returned
	// iterator will contain no keys.
	GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)
	// GetQueryResult performs a "rich" query against a state database. It is only supported by state databases that support rich query,
	// such as CouchDB. An error will be returned if this method is called against a state database that does not support rich query.
	GetQueryResult(query string) (StateQueryIteratorInterface, error)
	// GetHistoryForKey returns a history of key values across time.
	GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)

	// Transient Data Management

	// GetTransient returns the value of the specified `key` from the transient store
	GetTransient(key string) ([]byte, error)

	// Chaincode Invocation

	// GetArgs returns the argument values from the invocation message
	GetArgs() [][]byte
	// GetStringArgs returns the argument values from the invocation message as strings
	GetStringArgs() []string
	// GetFunctionAndParameters returns the function name and parameters as bytes
	GetFunctionAndParameters() (string, []string)

	// Invokes another chaincode with the specified `chaincodeName`, `args

```

