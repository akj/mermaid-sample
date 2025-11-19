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

### Detailed Flow

```mermaid
flowchart TD
    Start([Start]) --> NewProjectScreen[/New Project Screen/]

    subgraph ProjectCreation["1️⃣ Project Creation"]
        NewProjectScreen --> PrepareProject[Prepare Project Data]
        PrepareProject --> IsInternal{Internal<br/>Type?}
        IsInternal -->|Yes| SetInternal[Set Internal Flag]
        IsInternal -->|No| CreateProject
        SetInternal --> CreateProject[(Create Project)]
    end

    subgraph GrantSetup["2️⃣ Grant Setup (Optional)"]
        CreateProject --> HasGrant{Grant<br/>Funded?}
        HasGrant -->|Yes| PrepareGrant[Prepare Grant Data]
        HasGrant -->|No| InitCounter
        PrepareGrant --> CreateGrant[(Create Project Grant)]
        CreateGrant --> InitCounter[Initialize Counter]
    end

    subgraph ResourceLoop["3️⃣ Resource Creation Loop"]
        InitCounter --> MoreResources{More<br/>Resources?}
        MoreResources -->|Yes| ResourceScreen[/Resource Details Screen/]

        ResourceScreen --> GetResource[Get Resource]
        GetResource --> PrepareResource[Prepare Resource Data]
        PrepareResource --> DupeCheck[Check for Duplicates]

        DupeCheck --> IsDupe{Duplicate?}
        IsDupe -->|Yes| ErrorScreen[/Error Screen/]
        ErrorScreen -.Back.-> ResourceScreen

        IsDupe -->|No| GrantFunded{Grant<br/>Funded?}
    end

    subgraph GrantFunding["3️⃣a Grant Funding Configuration"]
        GrantFunded -->|Yes| GetGrants[Get Project Grants]
        GetGrants --> HasGrants{Has<br/>Grants?}
        HasGrants -->|No| NoGrantsError[Set Error]
        NoGrantsError --> ErrorScreen

        HasGrants -->|Yes| GrantLoop{{For Each Grant}}
        GrantLoop --> GrantScreen[/Grant Funding Screen/]
        GrantScreen --> FundedByGrant{Fund By<br/>This Grant?}
        FundedByGrant -->|Yes| AddToCollection[Add to Collection]
        FundedByGrant -->|No| GrantLoop
        AddToCollection --> GrantLoop

        GrantLoop -->|Done| CreateResourceGrant[(Create Resource<br/>with Grants)]
        CreateResourceGrant --> TransformGrants[Transform Grant Data]
        TransformGrants --> CreateGrantLinks[(Bulk Create<br/>Grant Links)]
        CreateGrantLinks --> Increment
    end

    GrantFunded -->|No| CreateResource[(Create Resource)]
    CreateResource --> Increment[Increment Counter]

    Increment --> MoreResources

    subgraph Summary["4️⃣ Summary & Completion"]
        MoreResources -->|No| GetAllResources[Get All Resources]
        GetAllResources --> SummaryScreen[/Summary Screen/]
        SummaryScreen --> CreateMilestones{Create<br/>Milestones?}
        CreateMilestones -->|Yes| LaunchMilestones[Launch Milestone Flow]
        CreateMilestones -->|No| End
        LaunchMilestones --> End([End])
    end

    %% Styling
    classDef screenStyle fill:#E6E6FA,stroke:#333,stroke-width:2px
    classDef dbStyle fill:#87CEEB,stroke:#333,stroke-width:2px
    classDef loopStyle fill:#FFD700,stroke:#333,stroke-width:2px
    classDef errorStyle fill:#FFB6C1,stroke:#333,stroke-width:2px
    classDef startEnd fill:#90EE90,stroke:#333,stroke-width:2px

    class NewProjectScreen,ResourceScreen,GrantScreen,SummaryScreen screenStyle
    class CreateProject,CreateGrant,CreateResource,CreateResourceGrant,CreateGrantLinks dbStyle
    class GrantLoop loopStyle
    class ErrorScreen errorStyle
    class Start,End startEnd
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
