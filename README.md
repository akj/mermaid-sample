# CreateProject Flow Diagram

## Overview
Comprehensive project creation flow that guides users through creating a new project with optional grant funding and resource assignments.

## Flow Diagrams

### High-Level Overview
```mermaid
flowchart LR
    Start([Start]) --> Input[/Collect Project Info/]
    Input --> Create[(Create Project)]
    Create --> Grant{Grant<br/>Funded?}
    Grant -->|Yes| GrantSetup[(Setup Grant)]
    Grant -->|No| Resources
    GrantSetup --> Resources{Add<br/>Resources?}
    Resources -->|Yes| ResourceLoop[Resource Loop]
    Resources -->|No| Summary
    ResourceLoop --> Summary[/Show Summary/]
    Summary --> Milestones{Create<br/>Milestones?}
    Milestones -->|Yes| SubFlow[Launch Milestone Flow]
    Milestones -->|No| End
    SubFlow --> End([End])

    classDef userInput fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef database fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef process fill:#F0E68C,stroke:#333,stroke-width:2px

    class Input,Summary userInput
    class Create,GrantSetup database
    class ResourceLoop process
```

### Detailed Flow (by Phase)

#### Phase 1: Project Creation
```mermaid
flowchart LR
    Start([Start]) --> NewProjectScreen[/New Project Screen/]
    NewProjectScreen --> PrepareProject[Prepare Project Data]
    PrepareProject --> IsInternal{Internal<br/>Type?}
    IsInternal -->|Yes| SetInternal[Set Internal Flag]
    IsInternal -->|No| CreateProject
    SetInternal --> CreateProject[(Create Project)]
    CreateProject --> Phase2([To Phase 2:<br/>Grant Setup])

    classDef screenStyle fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef dbStyle fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px
    classDef transition fill:#FFE4B5,stroke:#333,stroke-width:2px

    class NewProjectScreen screenStyle
    class CreateProject dbStyle
    class Start,Phase2 startEnd
```

#### Phase 2: Grant Setup (Optional)
```mermaid
flowchart LR
    Phase1([From Phase 1]) --> HasGrant{Grant<br/>Funded?}
    HasGrant -->|Yes| PrepareGrant[Prepare Grant Data]
    HasGrant -->|No| InitCounter[Initialize Counter]
    PrepareGrant --> CreateGrant[(Create Project Grant)]
    CreateGrant --> InitCounter
    InitCounter --> Phase3([To Phase 3:<br/>Resource Loop])

    classDef dbStyle fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px
    classDef processStyle fill:#F0E68C,stroke:#333,stroke-width:2px

    class CreateGrant dbStyle
    class Phase1,Phase3 startEnd
    class PrepareGrant,InitCounter processStyle
```

#### Phase 3: Resource Creation Loop
```mermaid
flowchart TD
    Phase2([From Phase 2]) --> MoreResources{More<br/>Resources?}
    MoreResources -->|Yes| ResourceScreen[/Resource Details Screen/]
    MoreResources -->|No| Phase4([To Phase 4:<br/>Summary])

    ResourceScreen --> GetResource[Get Resource]
    GetResource --> PrepareResource[Prepare Resource Data]
    PrepareResource --> DupeCheck[Check for Duplicates]

    DupeCheck --> IsDupe{Duplicate?}
    IsDupe -->|Yes| ErrorScreen[/Error Screen/]
    ErrorScreen -.Back.-> ResourceScreen

    IsDupe -->|No| GrantFunded{Grant<br/>Funded?}
    GrantFunded -->|No| CreateResource[(Create Resource)]
    GrantFunded -->|Yes| GrantConfig[See Grant Funding Details Below]

    CreateResource --> Increment[Increment Counter]
    GrantConfig -.-> Increment
    Increment --> MoreResources

    classDef screenStyle fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef dbStyle fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef errorStyle fill:#FFB6C1,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px
    classDef processStyle fill:#F0E68C,stroke:#333,stroke-width:2px
    classDef noteStyle fill:#FFF8DC,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5

    class ResourceScreen,ErrorScreen screenStyle
    class CreateResource dbStyle
    class Phase2,Phase4 startEnd
    class GetResource,PrepareResource,DupeCheck,Increment processStyle
    class GrantConfig noteStyle
```

#### Phase 3a: Grant Funding Configuration (Detail)
This section expands the "Grant Funded? → Yes" path from Phase 3.

```mermaid
flowchart TD
    Start([Grant Funded = Yes]) --> GetGrants[Get Project Grants]
    GetGrants --> HasGrants{Has<br/>Grants?}
    HasGrants -->|No| NoGrantsError[Set Error]
    NoGrantsError --> ErrorScreen[/Error Screen/]

    HasGrants -->|Yes| GrantLoop{{For Each Grant}}
    GrantLoop --> GrantScreen[/Grant Funding Screen/]
    GrantScreen --> FundedByGrant{Fund By<br/>This Grant?}
    FundedByGrant -->|Yes| AddToCollection[Add to Collection]
    FundedByGrant -->|No| GrantLoop
    AddToCollection --> GrantLoop

    GrantLoop -->|Done| CreateResourceGrant[(Create Resource<br/>with Grants)]
    CreateResourceGrant --> TransformGrants[Transform Grant Data]
    TransformGrants --> CreateGrantLinks[(Bulk Create<br/>Grant Links)]
    CreateGrantLinks --> Return([Return to<br/>Resource Loop])

    classDef screenStyle fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef dbStyle fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef loopStyle fill:#FFD700,stroke:#333,stroke-width:2px
    classDef errorStyle fill:#FFB6C1,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px
    classDef processStyle fill:#F0E68C,stroke:#333,stroke-width:2px

    class GrantScreen,ErrorScreen screenStyle
    class CreateResourceGrant,CreateGrantLinks dbStyle
    class GrantLoop loopStyle
    class Start,Return startEnd
    class GetGrants,TransformGrants,AddToCollection processStyle
```

#### Phase 4: Summary & Completion
```mermaid
flowchart LR
    Phase3([From Phase 3]) --> GetAllResources[Get All Resources]
    GetAllResources --> SummaryScreen[/Summary Screen/]
    SummaryScreen --> CreateMilestones{Create<br/>Milestones?}
    CreateMilestones -->|Yes| LaunchMilestones[Launch Milestone Flow]
    CreateMilestones -->|No| End([End])
    LaunchMilestones --> End

    classDef screenStyle fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px
    classDef processStyle fill:#F0E68C,stroke:#333,stroke-width:2px

    class SummaryScreen screenStyle
    class Phase3,End startEnd
    class GetAllResources,LaunchMilestones processStyle
```

## Key Components

### Screens
1. **NewProjectScreen** - Main data entry with 6 sections:
   - Welcome (instructions)
   - Project Information (name, identifier, status, program, account, opportunity, invoice status, project type)
   - Key Dates (kickoff, deadline)
   - Description
   - Budget Information (hours estimate, expense estimate, hourly rate)
   - Grant Configuration (grant selection, adjusted coverage)
   - Resource Creation (checkbox to create resources, number of resources)

2. **ResourceDetailsScreen** - Resource data collection (repeats for each resource)

3. **GrantFundingScreen** - Grant funding configuration (appears for each grant when resource is grant-funded)

4. **SummaryScreen** - Final confirmation with project details and resource table (conditional visibility)

5. **ErrorScreen** - Generic error display with back navigation

### Decision Points
- **IsProjectTypeInternal** - Sets Internal checkbox if type is Internal
- **IsProjectFundedByGrant** - Determines if Project_Grant record should be created
- **MoreResourcesToCreate** - Loop control for resource creation (checks CreateProjectResources checkbox AND counter)
- **CheckForDuplicateProjectResources** - Prevents duplicate resource assignments
- **CheckIfFundedByGrants** - Routes to grant configuration or simple resource creation
- **CheckIfGrantsFound** - Validates grants exist before configuration
- **CheckIfResourceFundedByGrant** - Determines grant associations per resource
- **CheckIfCreateMilestones** - Launches milestone subflow if requested

### Record Operations
- **CreateProject** - Creates MPM4_BASE__Milestone1_Project__c record with estimate fields
- **CreateProjectGrant** - Creates Project_Grant__c junction record
- **CreateProjectResource** - Creates Project_Resource__c (non-grant-funded)
- **CreateProjectResourceWithGrants** - Creates Project_Resource__c (grant-funded)
- **CreateGrantProjectResourcesAfterLoop** - Bulk creates Grant_Project_Resource__c records

### Loops
- **LoopThroughGrants** - Iterates through project grants for resource grant associations

### Error Handling
All record operations have fault connectors that route to ErrorScreen with descriptive messages:
- Project creation failure
- Project grant creation failure
- Resource creation failure
- Grant project resource creation failure
- Duplicate check failure
- Grant lookup failure
- Resource lookup failure
