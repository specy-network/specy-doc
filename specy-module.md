# Specy network module

## Abstract

Specy Module is an on chain module implemented by Specy Network, responsible for the implementation of all on chain functions of Specy Network, including

- Executor Staking: Executor obtains permission to execute tasks through tokens on the pledge chain, and obtains a certain percentage of commission based on the service fee for executing tasks. 

- User Deposit: Users pay the service fees for Gas and Specy Network for automated task execution through pledged tokens. 

- Task management: Responsible for managing tasks, including task registration, update, cancellation, and other functions, while assisting in subsequent automated transaction validation and function calls

- Proof verification: Verify the effectiveness of automatically executing Task transactions. 

- Proxy: Based on task information, call the relevant Hook function to execute the task.



## Contents

- [State](##State)

  - [Total Staking](###TotalStaking)

  - [Executors](###Executors)

  - [CurrentExecutor](###CurrentExecutor)

  - [Params](###Params)

  - [Tasks](###Tasks)

  - [Deposits](###Deposits)

    

- State Transitions

  - Staking
  - UpdateCurrentExecutor

- Messages

  - MsgCreateTask
  - MsgCancelTask
  - MsgCreateExecutor
  - MsgExecutorDelegate
  - MsgExecutorUnDelegate
  - MsgDeposit
  - MsgUnDeposit
  - [MsgExecuteTask](###MsgExecuteTask)

- Begin-Block

- End-Block

  - How current executor update
  - Executor Set Changes

- Hooks

- Events

  - Msg's
  - End-Block

- Client

  - CLI



## State

### TotalStaking

​	The total of all executor staking.

- TotalStaking:String(totalStaking)->uint64



### Executors

- ​	Executor : byte(address)->ProtocolBuffer(Executor)

```protobuf
message Executor {
	//与executor服务相关联的链上账户地址
	string address=1; 
	//该executor质押的token数量
	cosmos.base.v1beta1.Coin staking=2;
	//executor运行所在sgx enclave的远程认证报告
    string iasAttestatioReport = 3; 
    //executor运行所在sgx enclave 公钥
    string enclavePK = 4; 
}
```



### CurrentExecutor

​	The executor currently has task execution permission, which will switch after the specified block interval (block_in_epoch) in params.



- CurrentExecutor:String("CurrentExecutor")->ProtocolBuffer(CurrentExecutor)

```protobuf
message CurrentExecutor {
	//On chain account address associated with the executor service
	string address=1; 
	//Switch the block height of the executor
	int64 height=2;
}
```

You can query the specific information of the executor in the collection through address, and the height is initially set to block in the Genesis block_ In_ Epoch.



### Params

​	

```protobuf
message Params{
	//executor 质押的token类型
	string bond_denom=1;
	//executor切换间隔
	int32 block_in_epoch=2;
	//最小的质押数量
	int64 min_staking_amount=3;
	//每次task执行的收费
	int64 price=4;
	//每个task的基础收费
	int64 base_price=5;
}
```



### Tasks

Tasks are divided into two types in terms of execution times: single execution and periodic execution. A single execution is performed at the specified time or under the specified circumstances, after which the task will end. Periodic execution is executed every time the agreed time is reached, and the task still exists after execution.

- Task:byte(Hash)->ProtocolBuffer(Task)

```protobuf
message Task{
	//task module name or contract address
	string contract_address=1;
	//module hook function or contract method
	string method=2;
	//以json的形式对对应的参数值进行预定义,如果有则预设，如果没有则空
	string calldata=3;
	//task 是否为单次执行
	bool single=4;
	string rule_file=5;
}
```

calldata说明举例
```json
{
	"params":[
		{"value":"canshudezhi"},
		{"value":},
	],
	"index":1
	
}
```
index指定参数中第几个值需要engine填充



### Deposits

用户需要在模块中提前deposit一定数量token用于支付gas费和手续费，deposit的金额将被send到模块账户中。在创建和执行任务时，按照params中定义的参数从模块账户转移至executor，并减少task所有者的deposit额度。deposit对应的token denom与params中设定的boned_denom相同。

- Deposit:address->ProtocolBuffer(Deposit)

```protobuf
message Deposit{
	//用户地址
	string address=1;
	//额度
	int64 amount=2;
}
```

## State Transitions

### Staking
### UpdateCurrentExecutor

## Messages

### MsgCreateTask

### MsgCreateTask
### MsgCancelTask
### MsgCreateExecutor
### MsgExecutorDelegate
### MsgExecutorUnDelegate
### MsgDeposit
### MsgUnDeposit
### MsgExecuteTask
该msg用于在满足条件时触发链上module对task的执行，请求参数包括：Task的hash（即id）、参数(calldata)两个部分
```shell
stargazed tx specy execute-task sakfdhkajffdhkjsafkjd "{\"params\":[\"value\":20]}"
```

## Begin-Block

## End-Block

## Hooks

## Events

### Msg's

- CreateTask:
创建Task时产生的event，executor需关注。
```go
ctx.EventManager().EmitEvents(sdk.Events{
		sdk.NewEvent(
			//event的类别为"create_task"
			types.EventTypeCreateTask,
			//任务的主键hash
			sdk.NewAttribute(types.AttributeKeyTaskHash, string(taskHash)),
			//合约地址或模块名称
			sdk.NewAttribute(types.AttributeKeyContractAddress, msg.ContractAddress),
			//task所需要调用的合约方法或模块hook函数
			sdk.NewAttribute(types.AttributeKeyMethod, msg.Method),
			//任务的schema,json形式
			sdk.NewAttribute(types.AttributeKeyCalldata, msg.Calldata),
			//任务是否为单次执行
			sdk.NewAttribute(types.AttributeKeySingle, strconv.FormatBool(msg.Single)),
			//约束内容
			sdk.NewAttribute(types.AttributeKeyRuleFile, msg.RuleFile),
			//task 创建者
			sdk.NewAttribute(types.AttributeKeyCreator, msg.Creator),
		),
	})
```

- CancleTask:
取消Task时产生的event，executor需关注。
```go
ctx.EventManager().EmitEvents(sdk.Events{
		sdk.NewEvent(
			//event的类别为"cancle_task"
			types.EventTypeCancleTask,
			//任务的主键hash
			sdk.NewAttribute(types.AttributeKeyTaskHash, string(taskHash)),
		),
	})
```

