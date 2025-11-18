# CreateProject Flow Diagram

## Overview
Comprehensive project creation flow that guides users through creating a new project with optional grant funding and resource assignments.

## Flow Diagram

```mermaid
flowchart TD
    Start([Start Flow]) --> NewProjectScreen[/NewProjectScreen:\nWelcome Section\nProject Information\nKey Dates\nDescription\nBudget Information\nGrant Configuration\nResource Creation/]

    NewProjectScreen --> PrepareProject[Prepare Project:\nAssign screen values to\nProject record variable]

    PrepareProject --> IsProjectTypeInternal{Is Project\nType Internal?}

    IsProjectTypeInternal -->|Yes| SetInternalTrue[Set Internal = True]
    IsProjectTypeInternal -->|No| CreateProject[(Create Project Record)]
    SetInternalTrue --> CreateProject

    CreateProject --> IsProjectFundedByGrant{Is Project\nFunded By Grant?}

    IsProjectFundedByGrant -->|Yes| PrepareProjectGrant[Prepare Project Grant:\nAssign Grant and Coverage]
    IsProjectFundedByGrant -->|No| ResetResourceCounter[Reset Resource Counter = 0]

    PrepareProjectGrant --> CreateProjectGrant[(Create Project_Grant Record)]
    CreateProjectGrant --> ResetResourceCounter

    ResetResourceCounter --> MoreResourcesToCreate{More Resources\nTo Create?}

    MoreResourcesToCreate -->|Yes:\nCreateProjectResources=true\nAND\nResourcesCreatedIndex\n< NumResourcesToCreate| SetDisplayResourceNumber[Set Display Resource Number:\nCurrentResourceNumber =\nResourcesCreatedIndex + 1]

    SetDisplayResourceNumber --> ResourceDetailsScreen[/Resource Details Screen:\nResource Lookup\nRole\nLogged Time Multiplier\nEmployment Type\nIs Funded By Grants?/]

    ResourceDetailsScreen --> GetResourceForName[Get Resource For Name:\nQuery Resource record by ID]

    GetResourceForName --> PrepareProjectResourceRecord[Prepare Project Resource Record:\nAssign all field values]

    PrepareProjectResourceRecord --> GetProjectResourcesDupeCheck[Get Project Resources\nDupe Check:\nQuery for existing resource]

    GetProjectResourcesDupeCheck --> CheckForDuplicateProjectResources{Duplicate\nResource Found?}

    CheckForDuplicateProjectResources -->|Yes| SetDuplicateErrorMessage[Set Duplicate Error Message]
    SetDuplicateErrorMessage --> ErrorScreen[/Error Screen:\nDisplay error message/]

    CheckForDuplicateProjectResources -->|No| CheckIfFundedByGrants{Is Resource\nFunded By Grants?}

    CheckIfFundedByGrants -->|Yes| GetProjectGrants[Get Project Grants:\nQuery Project_Grant records]

    GetProjectGrants --> CheckIfGrantsFound{Grants\nFound?}

    CheckIfGrantsFound -->|No| SetNoGrantsErrorMessage[Set No Grants Error Message]
    SetNoGrantsErrorMessage --> ErrorScreen

    CheckIfGrantsFound -->|Yes| LoopThroughGrants{{Loop Through Grants}}

    LoopThroughGrants -->|For Each Grant| GrantFundingScreen[/Grant Funding Screen:\nIs resource funded by this grant?\nGrant Designation\nProject Role Classification/]

    GrantFundingScreen --> CheckIfResourceFundedByGrant{Resource\nFunded By\nThis Grant?}

    CheckIfResourceFundedByGrant -->|Yes| PrepareGrantProjectResource[Prepare Grant Project Resource:\nAdd to TempGrantProjectResources\ncollection]

    CheckIfResourceFundedByGrant -->|No| LoopThroughGrants
    PrepareGrantProjectResource --> LoopThroughGrants

    LoopThroughGrants -->|After All Grants| CreateProjectResourceWithGrants[(Create Project Resource\nWith Grants)]

    CreateProjectResourceWithGrants --> TransformGrantProjectResources[Transform Grant Project Resources:\nAdd Project_Resource__c lookup\nto grant resources]

    TransformGrantProjectResources --> CreateGrantProjectResourcesAfterLoop[(Bulk Create\nGrant_Project_Resource records)]

    CreateGrantProjectResourcesAfterLoop --> IncrementResourceIndex[Increment Resource Index:\nResourcesCreatedIndex + 1]

    CheckIfFundedByGrants -->|No| CreateProjectResource[(Create Project Resource)]

    CreateProjectResource --> IncrementResourceIndex

    IncrementResourceIndex --> MoreResourcesToCreate

    MoreResourcesToCreate -->|No:\nCreateProjectResources=false\nOR\nResourcesCreatedIndex\n>= NumResourcesToCreate| GetCreatedProjectResources[Get Created Project Resources:\nQuery all Project_Resource records\nfor this project]

    GetCreatedProjectResources --> SummaryScreen[/Summary Screen:\nProject Details\nProject Resources Table\n'if CreateProjectResources=true'\nCreate Milestones Checkbox/]

    SummaryScreen --> CheckIfCreateMilestones{Create\nMilestones?}

    CheckIfCreateMilestones -->|Yes| LaunchAddMilestones[Launch Add Milestones\nSubflow]
    CheckIfCreateMilestones -->|No| End([End Flow])
    LaunchAddMilestones --> End

    ErrorScreen -.->|Allow Back| ResourceDetailsScreen

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style NewProjectScreen fill:#E6E6FA
    style ResourceDetailsScreen fill:#E6E6FA
    style GrantFundingScreen fill:#E6E6FA
    style SummaryScreen fill:#E6E6FA
    style ErrorScreen fill:#FFB6C1
    style CreateProject fill:#87CEEB
    style CreateProjectGrant fill:#87CEEB
    style CreateProjectResource fill:#87CEEB
    style CreateProjectResourceWithGrants fill:#87CEEB
    style CreateGrantProjectResourcesAfterLoop fill:#87CEEB
    style LoopThroughGrants fill:#FFD700
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
