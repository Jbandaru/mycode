**Consolidator Current Logic**
 
Input Parameters (from URL):
  billId: 32-character string
  rn1: 32-character string - random number
  rn2: 32-character string - random number
  part: used to identify interfaceID
  docId: passed but not directly used in current code
 
Validations:
  Length of billId, rn1, and rn2 must be exactly 32 characters
  If invalid, audit log must be written, and the process should terminate
 
Interface Check:
  If part is present:
    Validate part exists in EBILL_INTERFACE table as interface_id
  If invalid, log audit and terminate
 
Data Fetch Logic:
  If part is missing or empty:
    Look up the EAPOST_VALIDATEREDIRECTION table with billId, rn1, rn2
    Retrieve accountNumber and docId as newBillId(from flex1)
  If part is valid:
    Use EBILL_NOTIFICATION table with billId, rn1, rn2 to get accountNumber and docId as newBillId
 
PDF Retrieval:
  Retrieve pdfBillKey - Join BILL_MAIN and BILL_VOLUME tables using accountNumber and newBillId
  If pdfBillKey not found, log audit and terminate
  Retrieve PDF - Use CISDBManager.getInstance().getPDFBill(accountNumber, pdfBillKey) to get PDF byte stream
    byte[] pdfData = CISDBManager.getInstance().getPDFBill(accountNumber, pdfBillKey);   
  If retrieval fails, log audit and terminate
 
Return Response:
  fileName = "MyBill.pdf";
  HEADER_NAME = "P3P";
  HEADER_VALUE_CP = "CP=\"NOI DSP COR NID CUR OUR NOR\"";
  HEADER_VALUE_POLICYREF = "policyref=\"http://www.socalgas.com/residential/scg_eapost_p3p.xml\"";
  HEADER_VALUES = HEADER_VALUE_CP + "," + HEADER_VALUE_POLICYREF;
 
  response.setHeader("Content-Disposition","inline; filename=" + fileName );
  response.addHeader(HEADER_NAME, HEADER_VALUES);
  response.setContentType("application/pdf; name=" + fileName);
  OutputStream output = response.getOutputStream();
  output.write(pdfData);
 
