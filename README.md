# Cloud IP Address Ranges
A simple tool for importing IP and CIDR records from large and large-ish cloud or hosting providers and outputing them as a [SQLlite database](https://github.com/stclaird/cloudIPtoDB/releases/download/v1.0.0/cloudIP.sqlite3.db) and [clear text files](https://github.com/stclaird/cloudIPtoDB/tree/main/ipfiles). Currently, the project imports IP CIDR networks from the following providers:

| Provider                  | Method                       |
| ------------------------- | ---------------------------- |
| [Amazon Web Services (AWS)](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/aws-ips.ip.txt) | downloaded from provider url |
| [Google Cloud (GCP)	](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/goog.ip.txt)	| downloaded from provider url |
| [Digital Ocean (DO)	](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/digitalocean.ip.txt)	| downloaded from provider url |
| [Microsoft Azure (Azure)](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/azure-public-cloud.ip.txt)	| downloaded from provider url |
| [CloudFlare](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/cloudflare-ipv4.ip.txt)				| downloaded from provider url |
|[ Oracle Cloud	](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/oracle-public.ip.txt)			| downloaded from provider url |
| [Linode](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/linode.ip.txt)     				| downloaded from provider url |
| [IBM](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/ibm.ip.txt)						| downloaded from provider github page |
| [IBM/Softlayer	](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/softlayer-ibm.ip.txt)			| from ASN Prefix				|
| [GoDaddy](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/godaddy-AS26496.ip.txt)					| from ASN Prefix				|
| [A2Hosting](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/a2hosting.ip.txt)					| from ASN Prefix				|
| [Dreamhost](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/dreamhost-AS26347.ip.txt)					| from ASN Prefix				|
| [Vercel](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/vercel-aws.ip.txt)\AWS				| from ASN Prefix				|
| [Heroku](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/heroku-aws.ip.txt)\AWS				| from ASN Prefix				|
| [Alibaba](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/alibaba-AS45102.ip.txt)					| from ASN Prefix				|
| [Tencent](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/tencent-AS45090.ip.txt)					| from ASN Prefix				|

## NOTES:
- The IP/CIDR  info is extracted from official APIs and pages published by the providers themselves. Except when it isn't, as not all providers publish this information publicly - and when it isn't published, it is inferred and extracted from ASN prefix information. _Currently using the ASN info provided by the splendid folks at [hackertarget](https://hackertarget.com/)._ 

- I believe that extracing IP Cidr info from ASN to be less accurate, as some organisations use ASNs from a parent company which they may share with more than one seperate entity/organisation. 
For example, the ASN AS20473 that advertises IP Cidr prefixes for Vultr hosting  is actually registered "The Constant Company, LLC" and is shared by other orgs that seem distinct from Vultr. So I have left Vultr and other hosts with ASNs like this out for now.

- In the case of AWS, Vercel and Heroku - Vercel and Heroku are hosted by AWS. So processing the provided [AWS IP list](https://github.com/stclaird/cloudIPtoDB/blob/main/ipfiles/aws-ips.ip.txt) should be enough, or so you might think. However, there are additional prefixes advertised in ASNs that Vercel and Heroku use that are not on the main AWS list.  

# Technology Stack

The two core elements of this project are:
 - A binary written in GoLang which creates a SQLite database object and populates it with IP CIDR data from various Cloud platform providers.
 - A SQLite database file output containing the Cloud platform provider's CIDR information.

The SQLite database schema is made up of a single 'net' table.  

```CREATE TABLE IF NOT EXISTS net (
 	net_id INTEGER PRIMARY KEY,
 	net TEXT NOT NULL,
 	start_ip INT NOT NULL,
 	end_ip INT NOT NULL,
 	url TEXT NOT NULL,
 	cloudplatform TEXT NOT NULL,
 	iptype TEXT NOT NULL
 	);
 ```
# Use the database for local querying

These instructions describe how to use the sqlite file on your workstation for local querying

## Method 1
The easiest method, ensure you have [SQLite](https://www.sqlite.org/download.html) installed and then simply download the latest cloudIP.sqlite3.db file from the releases section.

[https://github.com/stclaird/Cloud-IP-Address-Ranges/releases/latest](https://github.com/stclaird/Cloud-IP-Address-Ranges/releases/latest)

Then from the command prompt run the sqlite command and specify the newly download file path.

```
sqlite3 cloudIP.sqlite3.db
```

## Method 2
Use this version to build the binary and create the sqllite database file with the latest cloud provider CIDR information.  This method is slightly more complex but produces the most up to date database file.

Check out this repository
```
git clone https://github.com/stclaird/Cloud-IP-Address-Ranges)
```
Change directory to the cmd folder
```
cd Cloud-IP-Address-Ranges/cmd/main
```
Build the binary
```
go build -o cloudiptodb main.go
```

Run the binary
```
chmod +x cloudiptodb
./cloudiptodb
```

This will produce a database file in the directory /output
You can run this simply with the following command

```
sqlite3 output/cloudIP.sqlite3.db
```

# Querying the Database
Once you have the database open in SQLite then the following section gives examples on how to query the database using SQL statements.

### 1. To get the total number of CIDR records across all cloud providers held in the database:

```
select count(*) from net;
```

### 2. To get the number of CIDR records held in the database that belong to the cloud platform AWS.

```
select count(*) from net where cloudplatform='aws';
```

### 3. To get the CIDR records from cloudflare
```
select * from net where cloudplatform='cloudflare';
```
```
1729491968|103.21.244.0/22|1729491968|1729492991|https://www.cloudflare.com/ips-v4|cloudflare|IPv4
<MANY MORE RECORDS SNIP>
sqlite>
```

### 4. Find if a specific IP address exists in one of the cidrs held in the database.

The records in the database are in CIDR network format, and not unpacked and stored as individual IPv4 addresses.
Unpacking the CIDRs into a individual IP records generally doesn't make sense as it would be time consuming to unpack them all and create a much larger than necessary database.
However, the downside of not having individual IP addresses stored as seperate records is it does make querying for specific IPs using SQL difficult.

To remedy this and allow for the querying of individual IP addresses, we also store some additional columns along side the CIDR record, the start (network) and end (broadcast) address. These records are both stored as integers and this means we are able to query whether a specific IP address record exists in the database by testing if the IP address falls between the start record and the end record.

One thing though, for this to work you do need to convert your IP address to an integer before running a query.
For example, if you want to know if the IP address `177.71.207.129` is within one of the CIDR records stored in the database:

Firstly, you need to convert this IPv4 address to a decimal integer, which is 2974273409. Once you have the IP as an integer you can then perform the following query, where the value for the start_ip and end_ip columns are this integer:

```
SELECT cloudplatform, net
FROM net
WHERE start_ip <= '2974273409'
AND end_ip >= '2974273409';
```
If this IP address is contained within one of the CIDR records, this query will return a CIDR record similar to the following,

```
aws|177.71.128.0/17
aws|177.71.207.128/26
```
otherwise, if the IP address is not stored then the database will return no records.


