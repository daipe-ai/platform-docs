# Using the @notebook_function() decorator

The `@notebook_function()` decorator is very simple. It recieves any number of arguments, passes it down to its decorated function and runs the function. You can use it for example on a function which downloads data and you want to log its progress. 

```python
@notebook_function()
def download_data(logger: Logger):
    opener = urllib.request.URLopener()
    opener.addheader(
        "User-Agent",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36",
    )

    opener.retrieve("https://www.bondora.com/marketing/media/LoanData.zip", "/loanData.zip")
    logger.info("Loans data successfully downloaded")
    opener.retrieve("https://www.bondora.com/marketing/media/RepaymentsData.zip", "/repaymentsData.zip")
    logger.info("Repayments data successfully downloaded")
```
  
!!! info "Technical Reference"
    Check the [Technical reference](input-decorators.md#notebook_function) for more details about the @notebook_function and other decorators.
