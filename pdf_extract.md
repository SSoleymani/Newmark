---
title: "Extracting and Processing Property Information from PDFs"
output: 
  html_document: 
    code_folding: show
    keep_md: true
---



## Introduction

This document demonstrates how to extract text from a PDF document, process the extracted text using the OpenAI GPT-4 API, and format the extracted attributes into a structured JSON file. The process involves several steps, including loading necessary libraries, extracting text from PDFs, formatting the extracted text, sending it to the OpenAI API, parsing the response, and saving the results.

## Step 1: Extract Text from PDF

First, we load the necessary libraries and set the path to the PDF file. Then, we extract the text content from the PDF.


```r
# Load libraries
library(pdftools)
library(tidyverse)
```


```r
# Provide the path to the PDF file
pdf_file_path <- "https://stgendev01.blob.core.windows.net/python-test/nw1.pdf"

# Extract the text content from the PDF
text_content <- pdf_text(pdf_file_path)
```

## Step 2: Format Extracted Text for OpenAI API

We format the extracted text into JSON to prepare it for processing by the OpenAI API.


```r
text_content_json <- rjson::toJSON(text_content)
```

## Step 3: Send Text to OpenAI API

We construct the request body and send it to the OpenAI API to extract structured information from the text.




```r
url <- "https://api.openai.com/v1/chat/completions"


headers <- c(
  'Authorization' = paste('Bearer', YOUR_API_KEY),  # Replace YOUR_API_KEY with your actual key
  'Content-Type' = 'application/json'
)

body <- list(
  model = "gpt-4-1106-preview",
  messages = list(
    list(role = "system", content = "You are ChatGPT, a large language model trained by OpenAI."),
    list(role = "user", content = paste0("extract salient features of the property including the following for each property: Property:",
                                         "\n  - Address",
                                         "\n  - RentableArea",
                                         "\n  - NetOperatingIncome",
                                         "\n  - LandArea",
                                         "\n  - CapRate",
                                         "\n  - YearBuilt",
                                         "\n  - SalePrice",
                                         "\n  - Tenancy",
                                         "\n  - ExpirationDate",
                                         "\n  - TaxKey",
                                         "\n  - Taxes2021",
                                         "\n  - Municipality",
                                         "\n; as well as leasing details including:",
                                         "\n  - AnnualBaseRent",
                                         "\n  - CommonAreaMaintenance",
                                         "\n  - Insurance",
                                         "\n  - RealEstateTaxes",
                                         "\n  - ExpirationDate",
                                         "\n  - BaseRentIncrease",
                                         "\n  - Options",
                                         "\n  - Grantor",
                                         "\n and the Demographics as an Array for each Address",
                                         "\n from the pdf text below; DO NOT mention anything before and after the extracted data. just return the attributes in a format that I can easily parse to a structured JSON:\n", text_content_json))
  )
)

response <- POST(url, add_headers(.headers=headers), body = toJSON(body), encode = "json")
result <- content(response, as = "text", encoding = "UTF-8")
```

## Step 4: Parse API Response and Save as JSON

After receiving the response from the OpenAI API, we parse it and extract the relevant property information.


```r
content <- content(response, "parsed")
prop_info <- content$choices[[1]]$message$content
cat(prop_info)
```

```json
{
  "Properties": [
    {
      "Address": "1401 N Casaloma Dr, Appleton, WI",
      "RentableArea": "7,609 SF",
      "NetOperatingIncome": "$102,000",
      "LandArea": "1.32 AC",
      "CapRate": "6.75%",
      "YearBuilt": "1975",
      "SalePrice": "$1,511,111",
      "Tenancy": "Single",
      "ExpirationDate": "12-31-2033",
      "TaxKey": "101086805",
      "Taxes2021": "$12,099.79",
      "Municipality": "Grand Chute",
      "LeasingDetails": {
        "AnnualBaseRent": "$102,000 ($13.41PSF)",
        "CommonAreaMaintenance": "Paid directly by Tenant",
        "Insurance": "Paid directly by Tenant",
        "RealEstateTaxes": "Reimbursed monthly by Tenant",
        "ExpirationDate": "12-31-2033",
        "BaseRentIncrease": "$6,000 Annual Rent Increase Starting January 1st 2027",
        "Options": "Three (3) - Three (3) Year Options with 3% Annual Increases",
        "Grantor": "Brandon Fitness"
      },
      "Demographics": [
        {
          "Scope": "1 Mile",
          "Population": "1,978",
          "Households": "1,012",
          "MedianHHIncome": "$60,780",
          "Employees": "10,285"
        },
        {
          "Scope": "3 Miles",
          "Population": "39,799",
          "Households": "17,050",
          "MedianHHIncome": "$60,050",
          "Employees": "37,663"
        },
        {
          "Scope": "5 Miles",
          "Population": "112,732",
          "Households": "47,205",
          "MedianHHIncome": "$59,296",
          "Employees": "82,484"
        },
        {
          "Scope": "5 Minutes",
          "Population": "7,715",
          "Households": "3,623",
          "MedianHHIncome": "$69,971",
          "Employees": "21,473"
        },
        {
          "Scope": "10 Minutes",
          "Population": "83,808",
          "Households": "35,362",
          "MedianHHIncome": "$61,948",
          "Employees": "75,045"
        },
        {
          "Scope": "15 Minutes",
          "Population": "206,245",
          "Households": "85,173",
          "MedianHHIncome": "$61,533",
          "Employees": "137,747"
        }
      ]
    },
    {
      "Address": "W3171 Springfield Dr, Appleton, WI",
      "RentableArea": "6,528 SF",
      "NetOperatingIncome": "$102,000",
      "LandArea": "0.72 AC",
      "CapRate": "6.75%",
      "YearBuilt": "2017",
      "SalePrice": "$1,511,111",
      "Tenancy": "Single",
      "ExpirationDate": "12-31-2033",
      "TaxKey": "030264601",
      "Taxes2021": "$13,444.08",
      "Municipality": "Town of Buchanan",
      "LeasingDetails": {
        "AnnualBaseRent": "$102,000 ($15.63PSF)",
        "CommonAreaMaintenance": "Paid directly by Tenant",
        "Insurance": "Paid directly by Tenant",
        "RealEstateTaxes": "Reimbursed monthly by Tenant",
        "ExpirationDate": "12-31-2033",
        "BaseRentIncrease": "$6,000 Annual Rent Increase Starting January 1st 2027",
        "Options": "Three (3) - Three (3) Year Options with 3% Annual Increases",
        "Grantor": "Brandon Fitness"
      },
      "Demographics": [
        {
          "Scope": "1 Mile",
          "Population": "6,002",
          "Households": "2,237",
          "MedianHHIncome": "$84,176",
          "Employees": "5,797"
        },
        {
          "Scope": "3 Miles",
          "Population": "65,605",
          "Households": "25,244",
          "MedianHHIncome": "$66,012",
          "Employees": "24,258"
        },
        {
          "Scope": "5 Miles",
          "Population": "145,882",
          "Households": "59,080",
          "MedianHHIncome": "$60,615",
          "Employees": "74,795"
        },
        {
          "Scope": "5 Minutes",
          "Population": "22,866",
          "Households": "8,668",
          "MedianHHIncome": "$77,998",
          "Employees": "8,604"
        },
        {
          "Scope": "10 Minutes",
          "Population": "115,777",
          "Households": "46,174",
          "MedianHHIncome": "$62,705",
          "Employees": "59,831"
        },
        {
          "Scope": "15 Minutes",
          "Population": "215,292",
          "Households": "88,305",
          "MedianHHIncome": "$61,957",
          "Employees": "130,348"
        }
      ]
    }
  ]
}
```

Finally, we save the extracted information as a JSON file.


```r
jsonlite::write_json(prop_info, "output.json")
```

You can download the [extracted JSON file here](output.json).

## **Handling Scanned PDF Documents**

In this section, we process a PDF that is not based on text but rather on scanned images, such as photographs of documents. This requires a different approach from plain text extraction because we need to use Optical Character Recognition (OCR) to interpret the text from the images.

Similar to the previous section, we will use the **`pdftools`** library. However, since the PDF is more like a scanned picture, we will utilize the **`pdf_ocr_text`** function instead of **`pdf_text`** for text extraction:


```r
# Provide the path to the PDF file
pdf_file_path2 <- "https://stgendev01.blob.core.windows.net/python-test/BOV.pdf"

# Extract the text content from the PDF using OCR
text_content2 <- pdftools::pdf_ocr_text(pdf_file_path2)
```

```
## Converting page 1 to BOV_1.png... done!
## Converting page 2 to BOV_2.png... done!
## Converting page 3 to BOV_3.png... done!
## Converting page 4 to BOV_4.png... done!
## Converting page 5 to BOV_5.png... done!
## Converting page 6 to BOV_6.png... done!
## Converting page 7 to BOV_7.png... done!
## Converting page 8 to BOV_8.png... done!
## Converting page 9 to BOV_9.png... done!
## Converting page 10 to BOV_10.png... done!
## Converting page 11 to BOV_11.png... done!
## Converting page 12 to BOV_12.png... done!
## Converting page 13 to BOV_13.png... done!
## Converting page 14 to BOV_14.png... done!
## Converting page 15 to BOV_15.png... done!
```

```r
text_content_json2 <- rjson::toJSON(text_content2)
```


```r
body2 <- list(
  model = "gpt-4-1106-preview",
  messages = list(
    list(role = "system", content = "You are ChatGPT, a large language model trained by OpenAI."),
    list(role = "user", content = paste0("extract salient features of the property including the following for each property: Property:",
                                         "\n  - Address",
                                         "\n  - RentableArea",
                                         "\n  - NetOperatingIncome",
                                         "\n  - LandArea",
                                         "\n  - CapRate",
                                         "\n  - YearBuilt",
                                         "\n  - SalePrice",
                                         "\n  - Tenancy",
                                         "\n  - ExpirationDate",
                                         "\n  - TaxKey",
                                         "\n  - Taxes2021",
                                         "\n  - Municipality",
                                         "\n; as well as leasing details including:",
                                         "\n  - AnnualBaseRent",
                                         "\n  - CommonAreaMaintenance",
                                         "\n  - Insurance",
                                         "\n  - RealEstateTaxes",
                                         "\n  - ExpirationDate",
                                         "\n  - BaseRentIncrease",
                                         "\n  - Options",
                                         "\n  - Grantor",
                                         "\n and the Demographics as an Array for each Address",
                                         "\n from the pdf text below; DO NOT mention anything before and after the extracted data. just return the attributes in a format that I can easily parse to a structured JSON:\n", text_content_json2))
  )
)

response2 <- POST(url, add_headers(.headers=headers), body = toJSON(body2), encode = "json")
result2 <- content(response2, as = "text", encoding = "UTF-8")
```


```r
content2 <- content(response2, "parsed")
prop_info2 = content2$choices[[1]]$message$content
cat(prop_info2)
```

Finally, we save the extracted information as a JSON file.


```r
jsonlite::write_json(prop_info2, "output2.json")
```

You can download the [extracted JSON file](output2.json) of the second property.


## Conclusion

This R Markdown document demonstrated how to extract text from a PDF, process it with the OpenAI GPT-4 API, and save the processed information as a JSON file. By structuring unstructured data into a more digestible format, we can more easily analyze and utilize the information.
