# CreditRiskScoringStratio

## Business Case

> An important bank needs to ﬁlter good and bad credit requests as the bank receives them from potential clients (credit requesters). They need this to be automatic. The bank provides three ﬁles: 1) A sample of the bank’s historic credit request data, that they want to be automatically preprocessed from now on. 2) An example of one day’s requests to be used as input for the AI Model in the scoring workﬂow, and get a predicted default probability. 3) Credit bureau data (external data about clients) to enhance the scoring process with evidences of legal cases or police reports, applying this evidences of fraud in the workﬂow. Based on the data in this three ﬁles, the bank’s business analysts expect to receive a risk level scoring (red, yellow, or green) to be stored in a database (or parquet ﬁle) and presented in a Dashboard.

## Solution

> The workflow implemented for the business case can be found in the file 'workflow-batch-credit-risk-scoring-v0.json'. The overall of the design is a workflow where the main input is the example of the one day's requests data that is for the AI Model (riskscoring-v07), after some transformations the output is joined with the external auditory data of the clients, and then with some validations, clients are classified by their risk level scoring. There are two outputs, both in parquet files. One is for auditing before classification, and the other one contains the final data with the classifications. Each component of the design is explained below.

### CreditRequestsToday - (Input - Crossdata):

> This is the main input of the design. It gathers the data from the table client_credit_requests_today of the default database.

### MlModel - (Transformation - MlModel):

> The we execute the model riskscoring-v07 using the previous data as input.

### MlFiltered - (Transformation - Trigger):

> In here the resulting data after the execution of the model is transformed within a script in order to leave it with the same format that the historic data provided has (for dashboard purposes) and the probability value for the risk calculation is added as well.

### RenameColumns - (Transformation - RenameColumns):

> After the data was transformed some columns were Renamed for better visualization of the names and also to fit the name format that the Audit Data has (which later is going to be joined with).

### ExternalAuditoryData - (Input - Crossdata):

> It gathers the data from the table client_external_info of the default database.

### ProcessedAndAuditoryDataJoined - (Transformation - Join):

> It joins the data transformed and renamed from "RenameColumns" with the auditory data of the clients by the ID of the clients.

### DropDuplicatedID - (Transformation - DropColumns):

> It deletes the duplicated ID column resulting from the join.

## Dashboard

> The dashboard was intended to be done using BI in Discovery, however, the testing environment is presenting issues. The request was scaled to Stratio, and the response received indicated that the Dashboard is not necessary for this particular practice and certification.
