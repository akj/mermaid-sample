flowchart TD
    Start([Agent Opens Flow]) --> Search[Screen: Search/Select Customer]
    Search --> ExistingDecision{Existing<br/>Customer?}
    
    ExistingDecision -->|Yes| LoadAccount[Get Records:<br/>Retrieve Account]
    ExistingDecision -->|No| CreateCustomer[Screen:<br/>Create New Customer]
    
    CreateCustomer --> CreateAcctRecord[Create Records:<br/>New Account]
    CreateAcctRecord --> ServiceAddr
    LoadAccount --> ServiceAddr
    
    ServiceAddr[Screen:<br/>Enter Service Address] --> BillingAddr[Screen:<br/>Enter Billing Address]
    
    BillingAddr --> SameAddrDecision{Same as<br/>Service Address?}
    SameAddrDecision -->|Yes| CopyAddress[Assignment:<br/>Copy Service to Billing]
    SameAddrDecision -->|No| ManualBilling[Manual Entry]
    
    CopyAddress --> UpdateAddr
    ManualBilling --> UpdateAddr
    UpdateAddr[Update Records:<br/>Save Addresses to Account] --> GetProducts
    
    GetProducts[Get Records:<br/>Retrieve Available Products<br/>Filtered by Eligibility] --> CheckProducts{Products<br/>Available?}
    
    CheckProducts -->|No| ShowError[Display:<br/>No Products Available<br/>for Service Area]
    ShowError --> ServiceAddr
    CheckProducts -->|Yes| SelectProducts[Screen:<br/>Select Products<br/>Using Data Table]
    
    SelectProducts --> GetPricing[Get Records:<br/>Retrieve PricebookEntry<br/>for Selected Products]
    
    GetPricing --> StartLoop[Loop Start:<br/>For Each Selected Product]
    
    StartLoop --> LoadProduct[Assignment:<br/>Load Current Product Data]
    
    LoadProduct --> ConfigScreen[Screen:<br/>Configure Current Product<br/>Show Progress Indicator]
    
    ConfigScreen --> StoreConfig[Assignment:<br/>Store Configuration<br/>in Collection]
    
    StoreConfig --> LoopDecision{More<br/>Products?}
    
    LoopDecision -->|Yes| StartLoop
    LoopDecision -->|No| ReviewOrder[Screen:<br/>Review Order Summary]
    
    ReviewOrder --> ConfirmDecision{Confirm<br/>Order?}
    ConfirmDecision -->|No| SelectProducts
    ConfirmDecision -->|Yes| CreateOrder[Create Records:<br/>Create Order]
    
    CreateOrder --> OrderItemLoop[Loop:<br/>Through Product Collection]
    OrderItemLoop --> CreateOrderItem[Create Records:<br/>Create OrderItem]
    CreateOrderItem --> OrderItemLoop
    
    OrderItemLoop --> ContractDecision{Generate<br/>Service Agreement?}
    
    ContractDecision -->|Yes| CreateContract[Create Records:<br/>Create Contract]
    CreateContract --> PopulateContract[Assignment:<br/>Populate Contract Terms]
    PopulateContract --> PreviewContract[Screen:<br/>Preview Service Agreement]
    PreviewContract --> AcceptContract[Update Records:<br/>Mark Contract Accepted]
    AcceptContract --> Confirmation
    
    ContractDecision -->|No| Confirmation
    
    Confirmation[Screen:<br/>Display Order Confirmation] --> SendEmail[Send Email:<br/>Order Confirmation to Customer]
    SendEmail --> CreateTask[Create Records:<br/>Installation Follow-up Task]
    CreateTask --> End([Flow Complete])
    
    style Start fill:#e1f5e1
    style End fill:#ffe1e1
    style Search fill:#e3f2fd
    style SelectProducts fill:#e3f2fd
    style ConfigScreen fill:#e3f2fd
    style ReviewOrder fill:#e3f2fd
    style PreviewContract fill:#e3f2fd
    style Confirmation fill:#e3f2fd
    style CreateCustomer fill:#e3f2fd
    style ServiceAddr fill:#e3f2fd
    style BillingAddr fill:#e3f2fd
    style ExistingDecision fill:#fff3e0
    style SameAddrDecision fill:#fff3e0
    style CheckProducts fill:#fff3e0
    style ConfirmDecision fill:#fff3e0
    style ContractDecision fill:#fff3e0
    style LoopDecision fill:#fff3e0
    style CreateOrder fill:#f3e5f5
    style CreateOrderItem fill:#f3e5f5
    style CreateContract fill:#f3e5f5
    style CreateAcctRecord fill:#f3e5f5
    style UpdateAddr fill:#f3e5f5
    style GetPricing fill:#f3e5f5
    style GetProducts fill:#f3e5f5
    style LoadProduct fill:#f3e5f5
    style StoreConfig fill:#f3e5f5
    style StartLoop fill:#f3e5f5
