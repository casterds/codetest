# test

Employment Based Loan
This example is an implementation of a loan contract which is tied to a salary stream.

The application assumes 3 parties:
Employer
Borrower - an employee currently receiving a salary stream from the Employer
Lender - an outside party who lends to the Borrower
There are several steps involved in the origination of each loan contract:
The Borrower deploys a new EmploymentLoan.sol contract, and passes the following into the constructor:
borrow amount
the desired interest rate
the duration of the loan
amount of collateral posted
employer address
the borrower's own address
the token to be borrowed
the token to be locked as collateral
address of the Superfluid host contract on the current network
the address of the UniV3 pool to be used as a TWAP for checking collateral price
Once the contract is deployed, the borrower will instruct their employer to begin sending 100% of their salary into the contract. Because the application is a Super App, our contract will to run the afterAgreementCreated callback, where we have inserted logic which will initially send 100% of the incoming flow to the borrower. In this case, the borrower will still be receiving 100% of their salary.

Next, the borrower will deposit additional collateral into the contract if they have specified a collateral amount > 0 in the constructor of the deployed loan contract.

A lender may call approve() on the borrowToken contract, passing in the address of the current employmentLoan contract as the spender. From there, the lender may call the lend() function to enter into a loan agreement with the borrower. The lend function will ensure that there is the proper amount of collateral locked into the contract, and also check that the employer is streaming enough funds to cover interest payments on the loan. If both checks pass, then the lent funds will be sent to the borrower, and the flow from the contract to the employee will be split so that the lender is now receiving a streaming interest payment which is taken directly from the borrower's salary. The borrower's incoming salary stream is now equal to 100% of the previous salary amount - the lender's interest rate stream amount.

If the employer stops the flow rate into the employment loan contract or updates the flow rate below the amount needed to pay the lender's stream amount, then the flow from the contract to the lender and borrower is stopped. To keep the loan solvent, the contract will check the current price of the loan's collateral, calculate a flow rate which is enough to pay for the regular interest rate payments to the lender, and begin streaming the collateral token to the lender. If the contract at any point starts once again receiving a stream in the borrow token which is enough to pay for the lender's interest rate payments, the collateral token stream is stopped, and a new stream in the borrow token is started to both the lender and borrower once again as in step 4.

Once a loan is completed, the borrower may call closeLoan() to stop the stream to the lender, and once again receive 100% of their salary stream. A loan can be terminated early at any time by the borrower if the borrower is willing to pay off the remainder of the loan in a single transaction.
