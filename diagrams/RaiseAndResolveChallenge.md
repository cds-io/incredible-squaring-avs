
## RaiseAndResolveChallenge

A task challenger raises a challenge on a task response, and the task manager resolves the challenge. Here, is the flow of the challenge resolution process. It includes:

- Verify task response exists
- Check challenge is within time window
- Verify task hasn't been successfully challenged before
- Calculate actual squared output
- Compare with submitted response
- Get operator addresses from pubkey hashes
- Verify non-signing operators

**Successful Challenge**
- Mark task as successfully challenged
- Emit TaskChallengedSuccessfully event
- Potential slashing or freezing of validators (commented out in current version)

**Unsuccessful Challenge**
- Emit TaskChallengedUnsuccessfully event (Challenger receives no reward)

---

```mermaid
sequenceDiagram
    autonumber
    actor Challenger
    participant IncredibleSquaringTaskManager
    participant BLSApkRegistry
    actor Validators

    Challenger ->> IncredibleSquaringTaskManager: raiseAndResolveChallenge(task, taskResponse, taskResponseMetadata, pubkeysOfNonSigningOperators)
    activate IncredibleSquaringTaskManager

    IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Verify task response exists
    IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Check challenge is within time window
    IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Verify task hasn't been successfully challenged before

    IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Calculate actual squared output
    IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Compare with submitted response

    alt Incorrect Response, the submitted response is NOT the squared output of the task (Successful Challenge)
        IncredibleSquaringTaskManager ->> BLSApkRegistry: Get operator addresses from pubkey hashes
        activate BLSApkRegistry
        BLSApkRegistry -->> IncredibleSquaringTaskManager: operator addresses
        deactivate BLSApkRegistry
        
        IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Verify non-signing operators
        IncredibleSquaringTaskManager ->> IncredibleSquaringTaskManager: Mark task as successfully challenged
        IncredibleSquaringTaskManager -->> Challenger: Emit TaskChallengedSuccessfully event
        Note right of Challenger: Challenger potentially rewarded
        
        loop for each signing validator
            IncredibleSquaringTaskManager -->> Validators: Potential slashing or freezing (commented out in current version)
            activate Validators
            deactivate Validators
        end
    else Correct Response, the submitted response is the squared output of the task (Unsuccessful Challenge)
        IncredibleSquaringTaskManager -->> Challenger: Emit TaskChallengedUnsuccessfully event
        Note right of Challenger: Challenger receives no reward
    end

    deactivate IncredibleSquaringTaskManager
```