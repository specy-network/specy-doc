# Specy Engine
## Introduction
Specy Engine is the rule engine which used to run Specy Script Language and generate the compliance proof of task.

## Service

### 1. GetEngineInfo

**Service**

```protobuf
service Register{
    rpc GetEngineInfo (EngineInfoRequest) returns (EngineInfoResponse);
}
```

**Input Message**

```protobuf
message EngineInfoRequest {
    string ChainType = 1;
    string ChainId = 2;
}
```

**Output Message**
```protobuf
message EngineInfoResponse{
    string ias_attestation_report = 1; // remote attestation report receive from Intel IAS;
    string enclave_pk = 2; // the public key used to encrypt transaction;
    string error_info = 3;
}
```


### 2. GetTaskResult

**Service**

```protobuf
service Regulator {
    rpc GetTaskResult (stream TaskRequest) returns (stream TaskResponse) {}
}
```

**Input Message**

```protobuf
message TaskRequest {
    bytes taskhash = 1;
}
```

**Output Message**
```protobuf
message Result {
    bool status = 1;
    bytes task_result = 2; 
    string error_info = 3; // empty if status is true
}

message TaskResponse {
    bytes taskhash = 1;  
    Result result =2;  
    bytes rule_file_hash = 3;
    bytes signature = 4;
}
```