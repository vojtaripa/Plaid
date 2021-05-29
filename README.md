# Plaid
Plaid docs

Technical Assesment:

////////////////////////////////////////////////
Location: https://github.com/katherinemorgan-ut/TechnicalAssessment/blob/master/gistfile1.md


Please return your work as a gist and send to klehman@plaid.com

Note:

Use whatever resources you would like; Google, Stack Overflow, Plaid's documentation are all fair game.
Use whichever language you are most comfortable with.
Our work here on Support is collaborative, don't hesitate to ask questions!

Problem 1 

(SQL)


A major part of a Technical Support Engineer’s (TSE) role at Plaid is escalating customer reported issues to our Integrations team. When escalating an issue, TSEs are required to determine the number of Items that are affected by the issue for prioritization purposes. An Item is a set of credentials at a financial institution; each Item can have many Accounts associated with them. TSE’s scope issues like this by querying Plaid’s data warehouse using SQL (aka Simple Query Language).

*Jump to SQL_learning.md for a brief lesson on SQL*

Technical Support Engineers frequently scope the number of Items affected by a given issue using a table called airflow_collections.accounts_production

airflow_collections.accounts_production (table name):
item_id	account_id	name	institution_type	type	subtype	available_balance	current_balance
52a27332c0474ab823000003	52d558b3a7fabe103700000f	BofA Core Checking	amex	depository	checking	311.22	305.28
52a5520ccf2bc3b430000026	52d558b3a7fabe103700000e	CD	bofa	depository	cd	3825.24	47.26
52a639b77f1c1c63330004fd	530bfed37251c2fe17000014	American Express Gold Card	chase	credit	credit card	362.88	362.88
52d59675645592d31400000c	53549a7309f28b326b9e9fe9	Total Checking	wells	depository	savings	1410.75	1450
52d55790a7fabe103700000a	534da5027c4ba504370c0958	Money Market Savings	citi	depository	money market	86.75	309

Schema
Field Name	Type	Description
item_id	string	unique id assigned to each item
account_id	string	unique id assigned to each account
name	string	name of the account as it is returned by the institution
institution_type	string	abbreviated representation of the institutions name
type	string	primary account type categorization
subtype	string	secondary account categorization
available_balance	number	account balance that factors in funds allocated for pending transactions
current_balance	number	account balance that does not consider pending transactions
Each row represents one Plaid account and its associated details.

The following 4 cases are examples of data quality issues a TSE might encounter at Plaid. For each case, please provide the SQL query you would use to return the specified output:

A) You discover an issue with our Wells Fargo integration (institution_type: wells) that causes depository accounts to be returned with a null current balance. Return a list of account_ids affected by this issue.

A)SELECT account_ids 
  FROM airflow_collections.accounts_production 
  WHERE (institution_type like 'wells') AND (current_balance is NULL);

B) You discover a bug with our Chase integration (institution_type: chase) that causes us to return credit card accounts with the generic account name “CREDIT CARD”. Return the number of account_ids affected by this issue.

B)SELECT COUNT(account_ids)
  FROM airflow_collections.accounts_production
  WHERE institution_type like 'chase' AND name like 'CREDIT CARD';

C) You discover an issue with our Bank of America integration (institution_type: bofa) that causes accounts with the subtype:cd to be returned with a negative available balance. 
Return a list of distinct item_ids affected by this issue.

C)SELECT available_balance < 0
  FROM airflow_collections.accounts_production
  WHERE institution_type like 'bofa' AND subtype like 'cd'

D) You discover an issue with our American Express integration (institution_type: amex) that causes some money market accounts to be classified as type: depository and subtype: savings (you can expect all money market accounts to contain the words “Money Market” somewhere in the account name). Return the number of distinct items affected.

D)SELECT DISTINCT institution_type, type, subtype
  FROM airflow_collections.accounts_production
  WHERE institution_type like 'amex' AND type like 'depository' AND subtype like 'savings' AND name like '%Mondey Market%'

Problem 2 

(respond to customer)

A customer writes in with the following question:


Hi Plaid Support,

My customer linked their Wells Fargo account, account_id: 481032957, but their loan account is showing a balance of -10,513.00, which is the amount they owe. Your documentation says that amounts owed should be positive (see attached screenshot [at the bottom of this gist]).

Could you please look into why this is?

Matt


The attached file from Plaid's codebase, retrieveProducts.js, grabs the accounts for a given Item, determines what products should be retrieved for each account, and extracts any relevant data. Note that the functions to actually retrieve data are imported from another folder.

The specific account they added is of type loan.

Using the below file, retrieveProducts.js, as well as our documentation, determine why the balance for this account is wrong.

If there is a bug in the code, make a quick list in the text editor of your choice of what information might be useful for our Engineering team to investigate further or resolve the issue.

Once you have done so, write up a response to Matt's ticket in the same file.

MY ANSWER:

found in code in retrieveProducts.js file:

// all account types return a balance
  function balance(account) {
    let flippedAccountTypes = ['credit', 'loan'];

    if (flippedAccountTypes.includes(account.type)) {
       return Math.abs(account.balance);
    } else {
       return 0 - Math.abs(account.balance);
    }
  }

Those 2 should be switched. (I think) to get the positive value returned.

Problem 3

(API request and modifications)

A client has contacted Plaid Support with a couple of questions:

Are transaction amounts returned by Plaid numbers or strings?
Our users repeatedly ask us to return transactions of a specific amount during a given time frame - any ideas on how to do this?
In order to help this client out, please write a script that accomplishes the following:

 Performs an API request to Plaid's /transactions/get endpoint using a Sandbox access_token, that pulls transaction data from 2020-12-01 through 2020-12-31. (SHOULD BE 2021-04-15 to 2021-5-15)
 Consumes the JSON response from the above API request and determines whether amount fields are returned as a number or string. 
Checking one amount is sufficient.
Returns the transaction data for only transactions that are of the amount 6.33 during the above time period. 
Output can be either via the command line or as a file.

MY ANSWER:
use - https://plaid.com/docs/transactions/pagination/
and https://plaid.com/docs/api/products/#transactionsget
