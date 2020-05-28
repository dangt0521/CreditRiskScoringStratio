# CreditRiskScoringStratio

## Business Case

> An important bank needs to ﬁlter good and bad credit requests as the bank receives them from potential clients (credit requesters). They need this to be automatic. The bank provides three ﬁles: 1) A sample of the bank’s historic credit request data, that they want to be automatically preprocessed from now on. 2) An example of one day’s requests to be used as input for the AI Model in the scoring workﬂow, and get a predicted default probability. 3) Credit bureau data (external data about clients) to enhance the scoring process with evidence of legal cases or police reports, applying this evidences of fraud in the workﬂow. Based on the data in these three ﬁles, the bank’s business analysts expect to receive a risk level scoring (red, yellow, or green) to be stored in a database (or parquet ﬁle) and presented in a Dashboard.

## Solution

> The workflow implemented for the business case can be found in the file 'workflow-batch-credit-risk-scoring-v0.json'. The overall of the design is a workflow where the main input is the example of the one day's requests data that is for the AI Model (***riskscoring-v07***), after some transformations the output is joined with the external auditory data of the clients, and then with some validations, clients are classified by their risk level scoring. There are two outputs, both in parquet files. One is for auditing before classification, and the other one contains the final data with the classifications. Each component of the design is explained below.

### CreditRequestsToday - (Input - Crossdata):

> This is the main input of the design. It gathers the data from the table client_credit_requests_today of the default database.

### MlModel - (Transformation - MlModel):

> This executes the model ***riskscoring-v07*** using the previous data as input.

### MlFiltered - (Transformation - Trigger):

> Here the resulting data after the execution of the model is transformed within a script in order to leave it with the same format that the historic data provided has (for dashboard purposes) and the probability value for the risk calculation is added as well.

### RenameColumns - (Transformation - RenameColumns):

> After the data was transformed some columns were Renamed for better visualization of the names and also to fit the name format that the Audit Data has (which later is going to be joined with).

### ExternalAuditoryData - (Input - Crossdata):

> It gathers the data from the table client_external_info of the default database.

### ProcessedAndAuditoryDataJoined - (Transformation - Join):

> It joins the data transformed and renamed from "RenameColumns" with the auditory data of the clients by the ID of the clients.

### DropDuplicatedID - (Transformation - DropColumns):

> It deletes the duplicated ID column resulting from the join.

### Persist - (Transformation - Persist):

> In order to avoid useless computation overheads, the Persist operation is used to persist and/or cache the data set in memory across operations.

### ScoringModelAudit - (Output - Parquet):

> The data with the transformations before the classifications is saved in a parquet file on the path ***/certification/students/dangt0521/*** for auditing. The table name in crossdata for this data is ***ScoringModelAudit_dangt0521***.

### Repartition - (Transformation - Repartition):

> One partition was created for performance tuning.

### HighRiskFilter - (Transformation - Filter):

> This filter returns the data where the ProbabilityValue is minor than 0.8. If so, the client is classified with a ***red risk scoring level***.

### IncomeTaxAlert - (Transformation - Filter):

> With the discarded data of the previous filter, this filter returns the data where the applicant has not paid the income tax. If so, the client is classified with a ***red risk scoring level***.

### UnpaidCreditsOrFraudReports - (Transformation - Filter):

> With the discarded data of the previous filter, this filter returns the data where the applicant has been involved in a legal case involving unpaid credits or is registered in at least one police report related to fraud. If so, the client is classified with a ***red risk scoring level***.

### ExternalOrAuditDataFraud - (Transformation - Filter):

> With the discarded data of the previous filter, this filter returns the data where the applicant may be in a source of fraud audit data or has  been registered in any external data source as fraudulent when paying subscription services like Telecommunications, mobiles, etc. If so, the client is classified with a ***yellow risk scoring level***.

### NoFraudulentAddress - (Transformation - Filter):

> With the discarded data of the previous filter, this filter returns the data where the applicant has used false address fraudulently. If so, the client is classified with a ***yellow risk scoring level***.

### HighCreditSizeThreshold - (Transformation - Filter):

> With the discarded data of the previous filter, this filter returns the data where the credit amount of the applicant is greater than four thousand. If so, the client is classified with a ***green risk scoring level***.

### RedLevelScoring - (Transformation - Union):

> It unifies the data from the three red filters: HighRiskFilter, IncomeTaxAlert, and UnpaidCreditsOrFraudReports.

### YellowLevelScoring - (Transformation - Union):

> It unifies the data from the two yellow filters: ExternalOrAuditDataFraud, and NoFraudulentAddress.

### AddRedColumn - (Transformation - AddColumns):

> It adds the column RiskLevelScoring with the value ***Red*** for the data coming from the RedLevelScoring box.

### AddYellowColumn - (Transformation - AddColumns):

> It adds the column RiskLevelScoring with the value ***Yellow*** for the data coming from the YellowLevelScoring box.

### AddGreenColumn - (Transformation - AddColumns):

> It adds the column RiskLevelScoring with the value ***Green*** for the data coming from the HighCreditSizeThreshold filter box.

### CompleteRiskLevelScoring - (Transformation - Union):

> It unifies the data from the three AddColumns boxes: AddRedColumn, AddYellowColumn, and AddGreenColumn.

### DropColumns - (Transformation - DropColumns):

> It deletes the columns coming from the client_external_info table and the one with the probability value in order to preserve the historic client data format for dashboard purposes. The columns deleted are: AddressFraudCheck, ContactAudit, FraudSuspicion, LegalCase, PoliceReport, ProbabilityValue, and, UkvCheck.

### FinalRepartition - (Transformation - Repartition):

> One final partition was created for performance tuning.

### Parquet - (Output - Parquet):

> The data with all the transformations and classifications is saved in a parquet file on the path ***/certification/students/dangt0521/***. The table name in crossdata for this data is ***CreditRiskScoring_dangt0521***.

## Dashboard

> The dashboard was intended to be done using BI in Discovery, however, the testing environment is presenting issues. The request was scaled to Stratio, and the response received indicated that the Dashboard is not necessary for this particular practice and certification.
