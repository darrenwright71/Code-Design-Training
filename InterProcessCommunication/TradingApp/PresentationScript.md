Presentation Script
================


#### 0.	Preset VS solution. It has:

 - Infrastructure & Modules
 - Projects for modules and host
  
**Points:**
-	In Contracts only  interfaces and DTOs. No logic
-	No references between Module assemblies
    -   these are OperationContracts, DataContracts, FaultContracts


#### 1.	Create (code) the `IQuotationService` in `Contracts`

**Points:**
-	It�s an *System Contract* (any subsytem can use it)
-   Will be implemented by QuotationModule and used by other modules
-   Has to have only DTOs which can be serialized w/ many serializers (prefere array rather than abstract types)

#### 2. Implement the `QuotationService` in `QuotationModule`

```
class QuotationService : IQuotationService
    {
        private readonly Quotation[] array = 
            {
                new Quotation {AskPrice = 10.50m, BidPrice = 10.55m, SecurityCode = "ING.S.NYSE"},
                new Quotation {AskPrice = 12.50m, BidPrice = 12.55m, SecurityCode = "ING.B.NYSE"},
                new Quotation {AskPrice = 10.70m, BidPrice = 10.75m, SecurityCode = "AAPL.B.NASDAQ"},
                new Quotation {AskPrice = 11.50m, BidPrice = 11.55m, SecurityCode = "AAPL.S.NASDAQ"},
                new Quotation {AskPrice = 16.50m, BidPrice = 16.55m, SecurityCode = "MSFT.B.NASDAQ"},
                new Quotation {AskPrice = 17.50m, BidPrice = 17.55m, SecurityCode = "ING.B.AEX"},
                new Quotation {AskPrice = 10.51m, BidPrice = 10.59m, SecurityCode = "ING.S.AEX"},
            };
        
        public Quotation[] GetQuotations(string exchange, string instrument, DateTime @from, DateTime to)
        {
            var result = array.Where(q => q.SecurityCode.Contains(exchange));

            if (!string.IsNullOrWhiteSpace(instrument))
                result = result.Where(q => q.SecurityCode.Contains(instrument));

            return result.ToArray();
        }

        public Quotation[] GetQuotations(string securityCode, DateTime @from, DateTime to)
        {
            return array.Where(q => q.SecurityCode == securityCode).ToArray();
        }
    }
```

#### 3. Implement `OrderingService` in `SalesModule`


**Points:**
- Depends on the **contract** of the QuotationService NOT on the implementation
- No references among the modules. Look at the *Project Dependency Diagram*
- We need a way to link things at runtime

#### 4. Implement `PortfolioService` in `PortfolioModule`

**Points:**
- Depends on the **contract** of the QuotationService NOT on the implementation
- Its contract is also part of the `Contracts` assembly because it will be called from the UI, which will be hosted in a different process.
    - the point of this demo is not necessarely the communication between BE and FE, which might be done differently from the communication between BE components. 
    - however in most cases it is done in the same way, therefore it makes sens to add it in contracts.

#### 5. Introduce iQuarc.AppBoot

- Show that at this point the modules and services are not linked
  - they exist as libraries but no one loads / uses them
  - have a look on the *Project Dependencies Diagram*
- Install `iQuarc.AppBoot` for the projects in `\Modules\`
  - `iQuarc.AppBoot.Unity` is installed **only** in the host (the rest do not need to depend on Unity)
- The `ConsoleHost` be the process that will host the BE services 
  - in production this would be an web server (IIS) or a cloud service (Azure CloudService).
- We add a ConsoleUi for to demo how services are linked and used by the AppBoot.

#### 6. Bootstrapp iQuarc.AppBoot

- Setup the names of the assemblies that compose the app and need to be loaded by AppBoot
- Setup the projects to build in a upper `bin` folder, so they can be loaded in Debug env
- Decorate all implementations w/ `ServiceAttribute`
- Run ConsoleUi
  - Show how Portfolio service is called
  - Show how OrderingService is called

**Points**
 - No references between Modules, so we can deploy them on different processes / machines
 - For now, all are loaded in the same process w/ ConsoleUi
   - only function calls between them
   - Already advantages in Dependencies Management, Modularity and constraints for achieving Separtaion of Concerns
   - The deployment diagram for this would have ONE box: "The App"
     - Inside I have modularization and SoC, but everythig deploys on one box (an web server)
     - I may have scalability (if stateless) , availablity and high maintainability. Valid and OK scenario!