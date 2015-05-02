# Introduction #
In order to communicate with VeSPA, you can follow the steps below to configure and setup your client. In order to communicate, VeSPA uses XMLRPC, a remote procedure call function. Google protocol buffer is used to serialize, deserialize and the compression on the messages.

# Protocol Buffer #
Using Google Protocol-Buffer, one can communicate with the system in almost every language, as stated in <a href='https://developers.google.com/protocol-buffers/docs/reference/other'>official website</a>. Before start sending the requests to the PKI, follow this link to read more about Google Protocol Buffer:
<a href='https://developers.google.com/protocol-buffers/docs/tutorials'>Tutorials for Developers</a>. You can also add the plugin into your IDE in
<a href='http://plugins.netbeans.org/plugin/15724/protocol-buffers-netbeans-plugin'>Netbeans</a>, or <a href='https://code.google.com/p/protobuf-dt/'>Eclipse</a>. Since generating the classes from .proto file would happen only once, it suffices to generate it using the command. If it takes time to integrate the plugin, you can skip it.

**NOTE: For the installation of Xtext in Eclipse, ONLY install Xbase, Xtext Runtime and Xtext SDK of 2.4.2 version (instead of 2.3).**

The interfaces to the PKI is located: <a href='http://code.google.com/p/kth-vespa/source/browse/trunk/interfaces/interfaces.proto'>here</a>. Download this .proto file and compile the interface according to your desired language. You can compile and generate the corresponding classes either through the command, or the installed plugin in the IDE. Check <a href='https://developers.google.com/protocol-buffers/docs/tutorials'>Protocol Buffer Tutorial</a> for more information.


# Communication #
The communication to VeSPA is through XMLRPC, available for every programming language. The libraries can be downloaded from <a href='http://xmlrpc-c.sourceforge.net/'>xmlrpc-c</a> for C/C++, <a href='http://ws.apache.org/xmlrpc/'>Apache XML-RPC</a> for Java and <a href='https://code.google.com/p/android-xmlrpc/'>android-xmlrpc</a> for Android. A simple example is also availabe in <a href='http://sourceforge.net/p/xmlrpc-c/code/HEAD/tree/trunk/examples/cpp/sample_add_client_complex.cpp'>example</a>.

**NOTE: At the moment, please ignore certificate and host name verification (i.e., accept any) as illustrated in the following links:** <a href='http://stackoverflow.com/questions/12283237/ignore-certificate-verification-for-self-signed-certificate-in-c'>C/C++</a>, <a href='http://ws.apache.org/xmlrpc/ssl.html'>Java</a>, <a href='http://stackoverflow.com/a/9063005'>Android</a>



## Servers Details ##
LTCA, Long-Term Certificate Authority, is the responsible authority to issue voucher, ticket, and X.509 certificates. PCA, Pseudonymous Certificate Authority, is responsible for issuing pseudonymous certificates.

  * LTCA
    * 172.31.212.119
    * LTCA\_SERVER\_URL  = https://172.31.212.119/cgi-bin/ltca
    * LTCA\_METHOD\_NAME = ltca.operate


  * PCA
    * 172.31.212.102
    * PCA\_SERVER\_URL  = https://172.31.212.102/cgi-bin/pca
    * PCA\_METHOD\_NAME = pca.operate

  * RA
    * Private: 192.168.100.4
    * RA\_SERVER\_URL  = https://192.168.100.4/cgi-bin/ra
    * RA\_METHOD\_NAME = ra.operate

**The Root-CA's and servers' certificates are available**<a href='https://kth-vespa.googlecode.com/svn/trunk/Certificates/Certificates.zip'>certificates</a>.



&lt;BR&gt;


## LTCA Operational Code ##
The code numbers below show the request and response type to communicate with the servers. In each request and response, there is a field called: ReqType or ResType. The client identifies a valid Request-Type while the servers replies with the corresponding Response-Type. For example, if you want to receive a Voucher, the client sends 120 in the ReqType of the request while LTCA responses with 121 in the ResType.

```
REQ_VOUCHER_VEHICLE_TO_LTCA_USING_PROTO_BUFF                        120
RES_VOUCHER_LTCA_TO_VEHICLE_USING_PROTO_BUFF                        121
REQ_X509_CERT_REQ_VEHICLE_TO_LTCA_USING_PROTO_BUFF                  122
RES_ISSUE_X509_CERT_LTCA_TO_VEHICLE_USING_PROTO_BUFF                123
REQ_X509_CERT_VALIDATION_VEHICLE_TO_LTCA_USING_PROTO_BUFF           124
RES_X509_CERT_VALIDATION_LTCA_TO_VEHICLE_USING_PROTO_BUFF           125
REQ_NATIVE_TICKET_VEHICLE_TO_LTCA_USING_PROTO_BUFF                  126
RES_NATIVE_TICKET_LTCA_TO_VEHICLE_USING_PROTO_BUFF                  127
REQ_FOREIGN_TICKET_VEHICLE_TO_LTCA_USING_PROTO_BUFF                 128
RES_FOREIGN_TICKET_LTCA_TO_VEHICLE_USING_PROTO_BUFF                 129

```


## PCA Operational Code ##
```
REQ_NATIVE_PSNYM_CERT_VEHICLE_TO_PCA_USING_PROTO_BUFF               220
RES_NATIVE_PSNYM_CERT_PCA_TO_VEHICLE_USING_PROTO_BUFF               221
NATIVE_PSNYM_CERT_ACK_OK_VEHICLE_TO_PCA_USING_PROTO_BUFF            222
REQ_PSNYM_CRL_VEHICLE_TO_PCA_USING_PROTO_BUFF                       223
RES_PSNYM_CRL_PCA_TO_VEHICLE_USING_PROTO_BUFF                       224
REQ_UPGRADING_SYS_CONFIG_CLIENT_TO_PCA_USING_PROTO_BUFF             225
RES_UPGRADING_SYS_CONFIG_PCA_TO_CLIENT_USING_PROTO_BUFF             226
REQ_PSNYM_OCSP_VEHICLE_TO_PCA_USING_PROTO_BUFF                      227
RES_PSNYM_OCSP_PCA_TO_VEHICLE_USING_PROTO_BUFF                      228
```




&lt;BR&gt;


# How to make a request #
As demonstrated here, the protocol buffer message should be serialized and encoded in Base64 format before sending to the servers. The client should specify the first parameter in XMLRPC as ReqType while the second parameter as the serialized encoded protocol buffer message. The details of each field is pinpointed in <a href='http://code.google.com/p/kth-vespa/wiki/Tutorial'>Tutorial</a>.

## C/C++ APIs ##

```
interfaces::msgVoucherReq_V2LTCA stV2LTCA;

stV2LTCA.set_ireqtype(REQ_VOUCHER_VEHICLE_TO_LTCA_USING_PROTO_BUFF);
stV2LTCA.set_strusername(strUserName);
stV2LTCA.set_strpwd(strPwd);
stV2LTCA.set_stremailaddress(strEmailAddress);
stV2LTCA.set_strcaptcha(strCapcha);
iNonce = m_cmnUtils.getRnd();
stV2LTCA.set_inonce(iNonce);
stV2LTCA.set_ttimestamp(tTimeStamp);

std::string strSerialized;
if (!stV2LTCA.SerializeToString(&strSerialized)) {
       fprintf(stderr, "\nThe serialization failed on voucher request. "
}

strSerializedOutputData = encodeBase64(strSerialized, strSerialized.length());


...

xmlrpc_c::paramList serverMethodParams;
serverMethodParams.add(xmlrpc_c::value_int(iReqType));
serverMethodParams.add(xmlrpc_c::value_string(strSerializedOutputData));

...

```




&lt;BR&gt;


## Java APIs ##

```
msgVoucherReq_V2LTCA.Builder voucherReq = msgVoucherReq_V2LTCA.newBuilder();

voucherReq.setIReqType(reqType);
voucherReq.setStrUserName(strUserName);
voucherReq.setStrPwd(strPassword);
voucherReq.setStrEmailAddress(strEmailAddress);
voucherReq.setStrCaptcha(strCaptcha);
voucherReq.setINonce(iNonce);
voucherReq.setTTimeStamp(tTimeStamp);

String encodedReq = Base64.encodeToString(voucherReq.build().toByteArray(), Base64.NO_WRAP);

String response = (String) ltcaClient.call("ltca.operate", reqType, encodedReq);
```



&lt;BR&gt;


## Python APIs ##

```
```