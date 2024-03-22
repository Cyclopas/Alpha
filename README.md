<div align="center">

## Alpha

</div>



**Introduction:** The execution of transactional operations, particularly within blockchain networks, involves the initiation of fund transfers while ensuring the comprehensive sharing of transaction details with network nodes. This sharing encompasses both the primary transaction, denoted as Transaction A, and any prerequisite transactions it relies upon. The pivotal role of root transactions in connecting these antecedent transactions is emphasized.

**Root Transactions and Their Role:**  In this context, the concept of root transactions takes center stage. These central nodes establish a link between Transaction A and its antecedent transactions. The interconnectedness of these transactions emanates from the root transaction, providing a chronological and structural foundation.

**Merkle Root Computation and Identification:** A crucial facet of this system is the computation of the merkle root, facilitated by miners. This merkle root acts as a unique identifier for transactions within a block. Importantly, it encapsulates not only the transactions in the block but also includes the root transactions from which these transactions derive.

**Temporal Dimension and Historical Trajectory:** The inclusion of root transactions in multiple blocks imparts a temporal dimension to them. Each node within the blockchain architecture can trace the historical trajectory of a specific root transaction. Analyzing the sequence of blocks containing a particular root transaction reveals its frequency of use and recurrent application.

**Transactional Lineage Preservation:** Given this complex interplay, it becomes imperative to retain historical root transaction data for subsequent transactional engagements. Initiating new transactions based on a specific root transaction necessitates a comprehensive record of all prior transactions stemming from that root. The accuracy and integrity of this historical lineage serve as prerequisites for validating forthcoming transactions.

**Conclusion:** In the proposed method, nodes engage in block validation, a process where, post-validation, all transactions are discarded except coinbase transaction, leaving only the block headers and their associated Merkle trees intact. These Merkle trees store the hashes of every transaction duplicated within the block. Following this, nodes populate the mapTransactions data structure by extracting transaction hashes from these Merkle trees.

Consider a scenario where a transaction hash, denoted as 'A', is exclusively found within Block 1's Merkle tree and not within any other block's Merkle tree. This suggests that although 'A' is part of the network, it hasn't been utilized as an input in any transaction. Conversely, if 'A' is identified within Block 1 and 10, it signals that this hash has been spent in Block 10. Therefore, if a recipient wishes to spend more from this transaction hash, they must furnish the previous transaction where coins were spent in Block 10. Failure to provide this information will lead to rejection by nodes, as they are aware of the transaction's expenditure.
Nodes consistently adhere to the longest chain. If a dishonest node receives a payment in block x at transaction A and subsequently spends a portion of it in block z at transaction B, while attempting to erase transaction B from block z to deceive subsequent nodes, any alteration to a block will alter its hash. Therefore, recently joined nodes can detect this change reject the block.


The proposed method should not be misconstrued as superior to Bitcoin. Within a peer-to-peer network, nodes exhibit a dynamic nature, continuously joining and departing. They establish transaction indexes based on the blocks they receive from other nodes. For example, if Bob sends 30 coins to Alice in block 3 and a new node enters the network in block 10 ? questioning whether Bob indeed possessed 30 coins to spend. However, owing to the prevalence of honest nodes, the rest of the network can rely on their verifications. Additionally, unlike Bitcoin, which archives complete transaction records, a newly joined node can still authenticate the transaction.

Nodes consistently adhere to the longest chain principle. If a dishonest node receives a payment in block x at transaction A and subsequently utilizes a portion of it in block z at transaction B, attempting to erase transaction B from block z to deceive subsequent nodes, any modification to a block will result in an alteration of its hash. Consequently, recently joined nodes can detect this alteration and reject the block.

By simply adding a flag in the code, nodes can retain complete transaction records, enabling other nodes to query them in case of uncertainties regarding a transaction index. This serves as a secondary layer of verification, enhancing the robustness and reliability of the network.

#
#
#
#



                           +-------------------+
                           |   CTransaction    |
                           +-------------------+
                           | - nVersion: int   |
                     ------| - vin: CTxIn      |
                     |     | - vout: CTxOut    | ----------------------------------- |
                     |     | - nLockTime: int  |                                     |
                     |     |                   |                                     |
                     |     +-------------------+                                     |
                     |                                                               |
                     |                                                               v
                     |                                               +-------------------------------------+|
                     |                                               |             CTxOut                   |
                     |                                               +--------------------------------------+
                     |                                               | - int64: nValue                      |
                     |                                               | - scriptPubkey: CScript              |
                     v                                               +--------------------------------------+
      +--------------------------------------+        
      |             CTxIn                    |
      +--------------------------------------+      
      | - prevout: COutPrev                  |
      | - scriptSig: CScript                 |
      +--------------------------------------+
                       |
                       |
                       v
        +-----------------------------+
        |          COutPrev           |
        +-----------------------------+
        | - root: CTransaction        +
        | - prev: vector<CTransaction>|
        +-----------------------------+







#
#
#



```cpp
// Function to check if the input root transaction has sufficient coins for a transaction
bool checkBalance(CTransaction root) {
    // Calculate the total coins received in the input root transaction
    int64_t totalReceived = root.nValue;

    // Calculate the total coins spent by transactions linked to the input root transaction
    int64_t totalSpend = 0;
    for (tprev : root.vin.prevout.prev) {
        totalSpend += tprev.vout.nValue;
    }

    // Calculate the total balance remaining in the input root transaction
    int64_t totalBalance = totalReceived - totalSpend;

    // Check if the total balance is sufficient for the transaction
    return totalBalance >= root.nValue;
}



// Function to check the integrity of the input root transaction and its spending history
bool CheckRoot(CTransaction root) {
    // Compute the hash of the input root transaction
    uint256 hash = root.GetHash();
    
    // Ensure that the input root transaction exists in the transaction map
    if (!mapTransaction.count(hash)) {
        return false;
    }
    
    // Iterate through previous transactions spent by the input root transaction
    for (tprev : root.vin.prevout.prev) {
        // Compute the hash of the previous transaction
        uint256 pHash = txprev.GetHash();
        
        // Ensure that the previous transaction exists in the transaction map
        if (!mapTransaction.count(pHash)) {
            return false;
        }
        
        // Ensure that the previous transaction is linked to the input root transaction
        if (tprev.vin.prevout.root.GetHash() != hash) {
            return false;
        }
    }
    
    // Input root and its spending history are valid
    return true;
}


bool AcceptTransaction(CTransaction tx)
{
	uint256 hash = tx.GetHash();


	// check if already have 
    if (!mapTransaction.count(hash)) {
        return false;
    }


    // check root 
    if(!CheckRoot(tx.vin.prevout.root))
    	return false;


    // check balance 
    if(!checkBalance(tx.vin.prevout.root))
    	return false;


    SCript scriptPubKey = tx.vin.prevout.root.vout.scriptPubKey;
    SCript ScriSig = tx.vin.ScriSig;

    // verify signature 
    //.....................................

    return true;

}
```
