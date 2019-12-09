import json
import boto3
import re


def lambda_handler(event, context):
    filePath= event;
    bad_chars = [';', ':', '!', "*", ']', "[" ,'"', "{" , "}" , "'",","]
    
    s3BucketName = "YOUR_BUCKET_NAME"
    documentName = filePath
    textract = boto3.client('textract')
    
    # Call Amazon Textract
    response = textract.detect_document_text(
        Document={
            'S3Object': {
                'Bucket': s3BucketName,
                'Name': documentName
            }
        })
    
    result=[]
    processedResult=""
    for item in response["Blocks"]:
        if item["BlockType"] == "WORD":
            result.append(item["Text"])
            element = item["Text"] + " "
            processedResult += element
        
    print (processedResult)
    res=[]
    amount=""
    
    str_Amount= str(re.findall('(?i)[invoice total|amount|total|balance]\s*?[due|amount]*\s*[$]\d{1,10}[.]\d{0,2}',processedResult))
    amount= str(re.findall('\$[^\]]+',str_Amount))
   
    
    
    amount = str(amount.split(' ')[-1])
    
    for i in bad_chars :
        amount = amount.replace(i, '')

   
    if len(amount)>0 :
        res.append(str(amount))
    else:
        res.append('')
    


    #Extracting invoice number and removing the bad characters
    
    str_Invoice = str(re.findall('(?i)invoice\s*[#|number|num]+[\s][\d|\w|_,-|A-Za-z]+',processedResult)) 
    result = str(str_Invoice.split(' ')[-1])
    
    if "IN" not in result:
        str_Invoice = str(re.findall('(?i)IN[V]*[0-9]*',processedResult)) 
        result = str(str_Invoice.split(' ')[-1])
        
    
    for bad_char in bad_chars :
        result = result.replace(bad_char, '')

    if len(result) >0 :
        res.append(result)
    else:
        res.append('')
     
     
     
    str_InvoiceDate=str(re.findall('(?i)invoice\s*date\s*\d{1,2}?.\d{1,2}?.\d{1,4}',processedResult)) 
    print (str_InvoiceDate)
    try:
        str_InvoiceDate = str(str_InvoiceDate.split(' ')[-1])    
    except:
        pass
    
    
    if len(str_InvoiceDate)<=2 :
         str_InvoiceDate=str(re.findall('(\d+[-/]\d+[-/]\d+)',processedResult))
         try:
            str_InvoiceDate = str(str_InvoiceDate.split(' ')[1])    
         except:
             pass
        
    try:    
        for i in bad_chars :
            str_InvoiceDate = str_InvoiceDate.replace(i, '')
    except:
        pass

   
    if len(str_InvoiceDate)>0 :
        res.append(str_InvoiceDate)
    else:
        res.append('')            
    

    return {
        'statusCode': 200,
        #'body': JSON.stringify(result)
        'body': res
        
    }
