```mermaid
sequenceDiagram
    autonumber
    participant TR as Trainer PPO
    participant PA as PlayerAgent
    participant PB as PlayerBehavior
    participant PS as Projectile
    participant EN as Enemy
    participant PHY as Colliders Exit Finish Door

    TR->>PA: actions
    PA->>PB: Move and Aim
    alt Attack
        PA->>PB: Attack
        PB->>PS: Spawn projectile
        alt Hit Enemy
            PS->>EN: Damage
            EN-->>PA: Kill reward
        else Miss or hit wall or door
            PS-->>PA: Miss penalty
        end
    end
    alt Exit trigger
        PHY-->>PA: Exit
        PA-->>TR: done and clear reward
    else Finish collision
        PHY-->>PA: Finish
        PA-->>TR: penalty
    end
```

```mermaid
sequenceDiagram
    autonumber
    participant TR as mlagents-learn
    participant UE as Unity
    participant PA as PlayerAgent
    participant S as Sensors Ray and Grid
    participant W as World / Enemy / Door / Finish / Exit

    TR->>UE: Connect and start
    UE->>PA: OnEpisodeBegin reset
    loop Each step
        PA->>S: CollectObservations
        S-->>PA: obs
        PA-->>TR: obs reward done
        TR-->>PA: actions
        PA->>W: Move Aim Attack
        W-->>PA: events hit miss collision trigger
    end
    PA->>PA: AddReward or EndEpisode
```

```mermiad
flowchart LR
    UE[Unity Environment] --> O[Observations]
    O --> COM[ML-Agents Communicator]
    COM --> TR[Python Trainer mlagents-learn]

    TR --> A[Actions]
    A --> COM
    COM --> UE

    subgraph UnitySide[Unity Runtime Components]
        AG[PlayerAgent C#]
        PB[PlayerBehavior C#]
        EC[EnvironmentController C#]
        GM[GameManager C#]
        EN[NPC_Enemy C#]
        PR[Projectile C#]
        SS[Ray Sensor]
        GS[Grid Sensor]
        PHY[Physics Colliders Triggers]
        UE --> AG
        AG --> PB
        AG --> SS
        AG --> GS
        AG --> PHY
        EC --> EN
        EC --> GM
        PB --> PR
        PR --> EN
        EN --> GM
    end

    subgraph TrainerSide[Training Pipeline]
        PPO[PPO Update]
        BUF[Rollout Buffer]
        NET[Policy and Value Networks]
        TR --> BUF
        BUF --> PPO
        PPO --> NET
        NET --> TR
    end

    subgraph Outputs[Artifacts and Reporting]
        TB[TensorBoard Logs]
        CKPT[Checkpoints]
        ONNX[Export ONNX Model]
        CSV[Optional CSV Episode Logs]
        TR --> TB
        TR --> CKPT
        TR --> ONNX
        AG --> CSV
    end

    ONNX --> UE
```
