**Any comments for the judge to contextualise your findings**
My findings are as follows:
Reentrancy.
Dangerous math formula.
Burn vulnerability.
Denial of Service
Requirement violations.
**Approach taken in evaluating the codebase**
Read through the code base for typical vulnerabilities.
Also ctrl +f to search for keywords that are common vulnerabilities.
Then write an impact analysis and PoC for the vulnerability.
**Architecture recommendations**
Utilise AI more.
**Codebase quality analysis**
The contracts use custom errors to provide more descriptive error messages. 
This can help with debugging and understanding why a transaction failed.
**Centralisation risks**
The contracts have a number of owner-only functions, which could be a centralisation risk if the owner account is compromised.
**Mechanism review**
The contracts use Access Control Manager (ACM) to control access to certain functions. 
Only the contracts' owner or an account with the appropriate permissions can call these functions.
**Systemic risks**
I think the potential of Denial of Service vulnerabilities should be considered. 

### Time spent:
35 hours