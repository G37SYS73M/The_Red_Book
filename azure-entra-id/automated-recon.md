# Automated Recon

#### ROAD Tools <a href="#road-tools" id="road-tools"></a>

* RoadRecon ([https://github.com/dirkjanm/ROADtools](https://github.com/dirkjanm/ROADtools)) is a tool for enumerating Entra ID environments!
* RoadRecon uses a different version '1.61-internal' of AADGraph API that provides more information.
* Enumeration using RoadRecon includes three steps
  * Authentication
  * Data Gathering
  * Data Exploration
* `roadrecon` supports username/password, access and refresh tokens, device code flow (sign-in from another device) and PRT cookie.

```batch
cd C:\\AzAD\\Tools\\ROADTools
.\\venv\\Scripts\\activate
roadrecon auth -u [test@defcorphq.onmicrosoft.com](<mailto:test@defcorphq.onmicrosoft.com>) -p V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd

roadrecon gather 
roadrecon plugin policies 
roadrecon gui
```

#### StormSpotter - Dated <a href="#stormspotter-dated" id="stormspotter-dated"></a>

* StormSpotter ([https://github.com/Azure/Stormspotter](https://github.com/Azure/Stormspotter)) is a tool from Microsoft for creating attack graphs of Azure resources.
* It uses the Neo4j graph database to create graphs for relationships in Azure and Entra ID!
* It has following modules
  * Backend – This is used for ingesting the data in the Neo4j database
  * Frontend (WebApp) – This is the UI used for visualizing the data.
  * Collector – This is used to collect the data from Azure.
* Start the backend service

```batch
cd C:\\AzAD\\Tools\\stormspotter\\backend\\
pipenv shell
python ssbackend.pyz
```

* In a new process, Start the frontend webserver

```batch
cd C:\\AzAD\\Tools\\stormspotter\\frontend\\dist\\spa\\
quasar.cmd serve -p 9091 --history
```

* Use Stormcollector to collect the data.

```batch
cd C:\\AzAD\\Tools\\stormspotter\\stormcollector\\
pipenv shell
az login -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd
python C:\\AzAD\\Tools\\stormspotter\\stormcollector\\sscollector.pyz cli
```

* Log-on to the webserver at[http://localhost:9091](http://localhost:9091/) using the following: Username: neo4j Password: BloodHound Server: bolt://localhost:7687

#### BloodHound <a href="#bloodhound" id="bloodhound"></a>

```powershell
# Run the collector to gather data
$passwd = ConvertTo-SecureString "SuperVeryEasytoGuessPassword@1234" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential("test@defcorphq.onmicrosoft.com", $passwd) 
Connect-AzAccount -Credential $creds 
Connect-AzureAD -Credential $creds 
. C:\\AzAD\\Tools\\AzureHound\\AzureHound.ps1
Invoke-AzureHound -Verbose
```

* Find all users who have the Global Administrator role `MATCH p =(n)-[r:AZGlobalAdmin*1..]->(m) RETURN p`
* Find all paths to an Azure VM `MATCH p = (n)-[r]->(g: AZVM) RETURN p`
* Find all paths to an Azure KeyVault `MATCH p = (n)-[r]->(g:AZKeyVault) RETURN p`
* Find all paths to an Azure Resource Group `MATCH p = (n)-[r]->(g:AZResourceGroup) RETURN p`
* Find Owners of Entra ID Groups `MATCH p = (n)-[r:AZOwns]->(g:AZGroup) RETURN p`
* [https://github.com/LuemmelSec/Custom-BloodHound-Queries](https://github.com/LuemmelSec/Custom-BloodHound-Queries)
* [https://hausec.com/2020/11/23/azurehound-cypher-cheatsheet/](https://hausec.com/2020/11/23/azurehound-cypher-cheatsheet/)
