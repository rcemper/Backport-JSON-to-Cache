# Backport %JSON.* to Caché  
Full backport from IRIS for Windows (x86-64) 2020.1 (Build 215U) Mon Mar 30 2020 20:14:33 EDT

IRIS brought us an excellent %JSON.Package   
It is an essential component of the Project Manager (ZPM)  
This backport makes it available also in Caché and builds a base to eventually backport also ZPM.  

As writing to SYSLIB / IRISLIB is a strict NO-NO all .cls and .inc names now start with %Z*  
So %JSON.Adoptor became %ZJSON.Adaptor. 
Names of Properties & Methods inside the classes didn't change.
An extension of specific error codes was required as they don't exist in Caché  

### installation & content  ###  
As ZPM is not available (yet?) the traditional install of a package as .XML applies.  
It is all packed into %ZJSON.XML  
<?xml version="1.0" encoding="UTF-8"?>
<Export generator="Cache" version="25">
<Project name="%ZJSON" LastModified="2020-06-25 11:43:23.331591">
  <Items>
    <ProjectItem name="%ZJSON.Adaptor" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.Formatter" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.Generator" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.Mapping" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.MappingProperty" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.PropertyParameters" type="CLS"></ProjectItem>
    <ProjectItem name="%ZJSON.RawString" type="CLS"></ProjectItem>
    <ProjectItem name="%ZPVA.INC" type="MAC"></ProjectItem>
    <ProjectItem name="%ZjsonErrors.INC" type="MAC"></ProjectItem>
    <ProjectItem name="%ZjsonMap.INC" type="MAC"></ProjectItem>
  </Items>
</Project>
</Export>

I just tested it in Namespace "%SYS" with Studio import + compile   

### examples ###

In Namespace SAMPLES I extended
- Class Sample.Address Extends (%SerialObject, %Populate, %XML.Adaptor, __%ZJSON.Adaptor__ )  
- Class Sample.Person Extends (%Persistent, %Populate, %XML.Adaptor, __%ZJSON.Adaptor__ )
- recompiled Class Sample.Employee Extends Person
- Class Sample.Company Extends (%Persistent, %Populate, %XML.Adaptor, __%ZJSON.Adaptor__)

### quick test ###
single object
~~~
USER>zN "SAMPLES"
 
SAMPLES>do ##class(Sample.Person).%OpenId(3).%JSONExportToString(.jsn) w !,jsn,!
 
{"Name":"Eno,Ashley T.","SSN":"655-23-7554","DOB":"1957-10-05","Home":{"Street":"4763 Madison Place","City":"Elmhurst","State":"AR","Zip":"44523"},"Office":{"Street":"6551 First Blvd","City":"Chicago","State":"WV","Zip":"12600"},"FavoriteColors":["Red","Orange"],"Age":"62"}
 
SAMPLES>do ##class(%ZJSON.Formatter).%New().Format(jsn)
{
  "Name":"Eno,Ashley T.",
  "SSN":"655-23-7554",
  "DOB":"1957-10-05",
  "Home":{
    "Street":"4763 Madison Place",
    "City":"Elmhurst",
    "State":"AR",
    "Zip":"44523"
  },
  "Office":{
    "Street":"6551 First Blvd",
    "City":"Chicago",
    "State":"WV",
    "Zip":"12600"
  },
  "FavoriteColors":[
    "Red",
    "Orange"
  ],
  "Age":"62"
}
SAMPLES>
~~~
complex object
~~~
 
SAMPLES>do ##class(Sample.Employee).%OpenId(103).Company.%JSONExportToString(.jsn) w !,jsn,!! do ##class(%ZJSON.Formatter).%New().Format(jsn)
 
{"Name":"TeraData Partners","Mission":"Providers of high-tech predictive analytic XML gaming for high-worth individuals.","TaxID":"G3790","Revenue":"427485809","Employees":[{"Name":"Lennon,Xavier B.","SSN":"949-36-7517","DOB":"2011-03-02","Home":{"Street":"7251 Madison Avenue","City":"Vail","State":"KS","Zip":"12629"},"Office":{"Street":"9419 Clinton Drive","City":"Jackson","State":"IL","Zip":"95635"},"Spouse":{"Name":"Gaboriault,Tara K.","SSN":"196-41-3278","DOB":"2015-06-16","Home":{"Street":"4435 Second Place","City":"Hialeah","State":"VA","Zip":"37962"},"Office":{"Street":"3237 Washington Avenue","City":"Tampa","State":"AR","Zip":"91726"},"FavoriteColors":["White","Red"],"Age":"5"},"Age":"9","Title":"Associate Product Manager","Salary":"59576","Notes":"Xavier used to work at OptiSystems Group Ltd. as a(n) Executive Systems Engineer","Picture":"iVBORw0KGgoAAAANSUhEUgAAABIAAAARCAIAAABfOGuuAAAAK3RFWHRDcmVhdGlvbiBUaW1lAFRodSA2IEphbiAyMDExIDE2OjIwOjU4IC0wNTAw73VlcAAAAAd0SU1FB9sBBhUWCKSIbF4AAAAJcEhZcwAACxIAAAsSAdLdfvwAAAAEZ0FNQQAAsY8L/GEFAAAC1klEQVR42l2TyW/TQBSHZ7MdO3HsZmubtKnUTaWlKggEhwoJOCAh9W/ljjhxAEERbWmhFaVruiXYbpw43sYzHhyo2N5tpPfNfPq9edB1XfBvCQGYEClgEEAEEIYIwv9aAPn7wAXwKLcSZidRhK9kFNRIroTKGjIVLP9N/8FiDo+9dJdefOODTlBNMFCQO19s3ZaVYtIw0fyoVpUw/geLGdiyo3fBJa+2GNZVovth3YnmfbaLzA/TeP3k+mKRPWoW6wSjG4yn4ND1Xtq7tGLN5+vjuAkU7ZMn7Qjk0LGDRJspxGp4+LFNVbw2ppcz2SHaC5P19n6PbFVMboDKglRZyuWbCtQIZ0hcc8VLSQ4ODu3XO1dfKGcZgrLcOp6353x1qR0kkZs6nTi2YuhSGFMEOIQCHThnb1utIOltnzh9f4gRxkXb7bcHXUuGzrlSkUunsiIhdhYC2xcRSFJGnU4TWflxbS/wW5YbVoo5wpjo9lLnfMrrjTJzqquVLxSIcUzTLKeEoYhxRbNLNWiDVFjWteP6YtIkAAIey0lXSXxCYxGP+EH2WBZXKlKW5KTew5nO4yVvAtq9LnmR2Qs6lCQYFBVZiiQQXwmaJoMRkJfBEEsh9RpTGyuFzhiEqQ8vLS5xtVRQsySJRFC9pEwaoaG+DXjx8HQp4qbASHCOmQvI/h6xNlxRkYTncQ1NVkZ0COFwbhM19eGcfjQIpMJJaF8ffZ7hQR6wNGXRaUd2jXzk4ca01xgtP19ZNgq5m3EbuvRgYfb0zaIVOM2x47DrXu7WWNvIPL3vWiJrmo5E1Vy4s3z31pws4xsMITjbrK7Rp6822Ym/XTZZqHKey0yRLBFZwmZZWV1Yenb/3i/DP38yu2NxukHgk/fbo5vn3zlJ6IjIdkZVSb2u373XWF2dmxwvDQP8WfD3vnHOgyDoe77jDKyO3+/HCAHDUKo1vVTSi0VNVVWE0K/mHyUqfH/CYKtlAAAAAElFTkSuQmCC"},{"Name":"Ipsen,Jules U.","SSN":"297-21-2682","DOB":"2009-10-20","Home":{"Street":"4291 Main Place","City":"Newton","State":"DE","Zip":"47093"},"Office":{"Street":"2019 Franklin Place","City":"Newton","State":"VA","Zip":"92649"},"Spouse":{"Name":"Adam,Jane C.","SSN":"970-42-3768","DOB":"1980-03-04","Home":{"Street":"1395 Maple Court","City":"Boston","State":"IL","Zip":"80443"},"Office":{"Street":"38 Franklin Avenue","City":"Oak Creek","State":"ME","Zip":"42689"},"FavoriteColors":["Purple"],"Age":"40"},"Age":"10","Title":"Strategic Accounts Rep.","Salary":"64695","Notes":"Jules used to work at BioNet LLC. as a(n) Global Developer","Picture":"iVBORw0KGgoAAAANSUhEUgAAABIAAAARCAIAAABfOGuuAAAAK3RFWHRDcmVhdGlvbiBUaW1lAFRodSA2IEphbiAyMDExIDE2OjIwOjU4IC0wNTAw73VlcAAAAAd0SU1FB9sBBhUWCKSIbF4AAAAJcEhZcwAACxIAAAsSAdLdfvwAAAAEZ0FNQQAAsY8L/GEFAAAC1klEQVR42l2TyW/TQBSHZ7MdO3HsZmubtKnUTaWlKggEhwoJOCAh9W/ljjhxAEERbWmhFaVruiXYbpw43sYzHhyo2N5tpPfNfPq9edB1XfBvCQGYEClgEEAEEIYIwv9aAPn7wAXwKLcSZidRhK9kFNRIroTKGjIVLP9N/8FiDo+9dJdefOODTlBNMFCQO19s3ZaVYtIw0fyoVpUw/geLGdiyo3fBJa+2GNZVovth3YnmfbaLzA/TeP3k+mKRPWoW6wSjG4yn4ND1Xtq7tGLN5+vjuAkU7ZMn7Qjk0LGDRJspxGp4+LFNVbw2ppcz2SHaC5P19n6PbFVMboDKglRZyuWbCtQIZ0hcc8VLSQ4ODu3XO1dfKGcZgrLcOp6353x1qR0kkZs6nTi2YuhSGFMEOIQCHThnb1utIOltnzh9f4gRxkXb7bcHXUuGzrlSkUunsiIhdhYC2xcRSFJGnU4TWflxbS/wW5YbVoo5wpjo9lLnfMrrjTJzqquVLxSIcUzTLKeEoYhxRbNLNWiDVFjWteP6YtIkAAIey0lXSXxCYxGP+EH2WBZXKlKW5KTew5nO4yVvAtq9LnmR2Qs6lCQYFBVZiiQQXwmaJoMRkJfBEEsh9RpTGyuFzhiEqQ8vLS5xtVRQsySJRFC9pEwaoaG+DXjx8HQp4qbASHCOmQvI/h6xNlxRkYTncQ1NVkZ0COFwbhM19eGcfjQIpMJJaF8ffZ7hQR6wNGXRaUd2jXzk4ca01xgtP19ZNgq5m3EbuvRgYfb0zaIVOM2x47DrXu7WWNvIPL3vWiJrmo5E1Vy4s3z31pws4xsMITjbrK7Rp6822Ym/XTZZqHKey0yRLBFZwmZZWV1Yenb/3i/DP38yu2NxukHgk/fbo5vn3zlJ6IjIdkZVSb2u373XWF2dmxwvDQP8WfD3vnHOgyDoe77jDKyO3+/HCAHDUKo1vVTSi0VNVVWE0K/mHyUqfH/CYKtlAAAAAElFTkSuQmCC"},{"Name":"Uhles,Clint S.","SSN":"469-62-1638","DOB":"1944-03-26","Home":{"Street":"3155 Elm Blvd","City":"Reston","State":"SC","Zip":"83447"},"Office":{"Street":"7476 Clinton Blvd","City":"Newton","State":"MS","Zip":"82343"},"Spouse":{"Name":"Paladino,Imelda S.","SSN":"355-51-9101","DOB":"2016-04-09","Home":{"Street":"6070 Main Place","City":"Fargo","State":"ID","Zip":"36015"},"Office":{"Street":"4662 Second Drive","City":"Albany","State":"VT","Zip":"41903"},"FavoriteColors":["Green","Green"],"Age":"4"},"FavoriteColors":["Yellow"],"Age":"76","Title":"Senior Marketing Manager","Salary":"488"},{"Name":"Evans,Zoe Y.","SSN":"836-46-4116","DOB":"1961-11-12","Home":{"Street":"5158 Second Street","City":"Newton","State":"CO","Zip":"36004"},"Office":{"Street":"5665 Main Avenue","City":"Tampa","State":"CA","Zip":"22666"},"Spouse":{"Name":"Chadwick,David D.","SSN":"867-20-2567","DOB":"1982-09-04","Home":{"Street":"1252 Franklin Avenue","City":"Zanesville","State":"LA","Zip":"74755"},"Office":{"Street":"9640 Washington Court","City":"Boston","State":"AZ","Zip":"61846"},"Age":"37"},"Age":"58","Title":"Assistant Systems Engineer","Salary":"58285"},{"Name":"Yancik,Roberta U.","SSN":"864-70-8242","DOB":"1998-01-12","Home":{"Street":"9652 Maple Court","City":"Chicago","State":"MS","Zip":"67796"},"Office":{"Street":"6014 Washington Court","City":"Ukiah","State":"WV","Zip":"56011"},"Spouse":{"Name":"Eagleman,Valery P.","SSN":"231-81-6984","DOB":"1996-12-25","Home":{"Street":"3118 Ash Street","City":"Miami","State":"MI","Zip":"84970"},"Office":{"Street":"3776 Clinton Blvd","City":"Ukiah","State":"CT","Zip":"57613"},"FavoriteColors":["Green"],"Age":"23"},"Age":"22","Title":"Resources Director","Salary":"75331"},{"Name":"Braam,Kim D.","SSN":"119-77-3119","DOB":"1954-01-17","Home":{"Street":"5808 Franklin Drive","City":"Elmhurst","State":"MD","Zip":"19151"},"Office":{"Street":"4002 Oak Street","City":"Xavier","State":"NH","Zip":"90182"},"Spouse":{"Name":"Ulman,Ralph Z.","SSN":"686-57-5448","DOB":"2014-07-24","Home":{"Street":"3482 Franklin Court","City":"Bensonhurst","State":"IL","Zip":"67956"},"Office":{"Street":"7438 Elm Street","City":"St Louis","State":"VT","Zip":"51597"},"FavoriteColors":["Blue","Black"],"Age":"5"},"FavoriteColors":["Orange"],"Age":"66","Title":"Laboratory Hygienist","Salary":"65936"}]}
 

{
  "Name":"TeraData Partners",
  "Mission":"Providers of high-tech predictive analytic XML gaming for high-worth individuals.",
  "TaxID":"G3790",
  "Revenue":"427485809",
  "Employees":[
    {
      "Name":"Lennon,Xavier B.",
      "SSN":"949-36-7517",
      "DOB":"2011-03-02",
      "Home":{
        "Street":"7251 Madison Avenue",
        "City":"Vail",
        "State":"KS",
        "Zip":"12629"
      },
      "Office":{
        "Street":"9419 Clinton Drive",
        "City":"Jackson",
        "State":"IL",
        "Zip":"95635"
      },
      "Spouse":{
        "Name":"Gaboriault,Tara K.",
        "SSN":"196-41-3278",
        "DOB":"2015-06-16",
        "Home":{
          "Street":"4435 Second Place",
          "City":"Hialeah",
          "State":"VA",
          "Zip":"37962"
        },
        "Office":{
          "Street":"3237 Washington Avenue",
          "City":"Tampa",
          "State":"AR",
          "Zip":"91726"
        },
        "FavoriteColors":[
          "White",
          "Red"
        ],
        "Age":"5"
      },
      "Age":"9",
      "Title":"Associate Product Manager",
      "Salary":"59576",
      "Notes":"Xavier used to work at OptiSystems Group Ltd. as a(n) Executive Systems Engineer",
      "Picture":"iVBORw0KGgoAAAANSUhEUgAAABIAAAARCAIAAABfOGuuAAAAK3RFWHRDcmVhdGlvbiBUaW1lAFRodSA2IEphbiAyMDExIDE2OjIwOjU4IC0wNTAw73VlcAAAAAd0SU1FB9sBBhUWCKSIbF4AAAAJcEhZcwAACxIAAAsSAdLdfvwAAAAEZ0FNQQAAsY8L/GEFAAAC1klEQVR42l2TyW/TQBSHZ7MdO3HsZmubtKnUTaWlKggEhwoJOCAh9W/ljjhxAEERbWmhFaVruiXYbpw43sYzHhyo2N5tpPfNfPq9edB1XfBvCQGYEClgEEAEEIYIwv9aAPn7wAXwKLcSZidRhK9kFNRIroTKGjIVLP9N/8FiDo+9dJdefOODTlBNMFCQO19s3ZaVYtIw0fyoVpUw/geLGdiyo3fBJa+2GNZVovth3YnmfbaLzA/TeP3k+mKRPWoW6wSjG4yn4ND1Xtq7tGLN5+vjuAkU7ZMn7Qjk0LGDRJspxGp4+LFNVbw2ppcz2SHaC5P19n6PbFVMboDKglRZyuWbCtQIZ0hcc8VLSQ4ODu3XO1dfKGcZgrLcOp6353x1qR0kkZs6nTi2YuhSGFMEOIQCHThnb1utIOltnzh9f4gRxkXb7bcHXUuGzrlSkUunsiIhdhYC2xcRSFJGnU4TWflxbS/wW5YbVoo5wpjo9lLnfMrrjTJzqquVLxSIcUzTLKeEoYhxRbNLNWiDVFjWteP6YtIkAAIey0lXSXxCYxGP+EH2WBZXKlKW5KTew5nO4yVvAtq9LnmR2Qs6lCQYFBVZiiQQXwmaJoMRkJfBEEsh9RpTGyuFzhiEqQ8vLS5xtVRQsySJRFC9pEwaoaG+DXjx8HQp4qbASHCOmQvI/h6xNlxRkYTncQ1NVkZ0COFwbhM19eGcfjQIpMJJaF8ffZ7hQR6wNGXRaUd2jXzk4ca01xgtP19ZNgq5m3EbuvRgYfb0zaIVOM2x47DrXu7WWNvIPL3vWiJrmo5E1Vy4s3z31pws4xsMITjbrK7Rp6822Ym/XTZZqHKey0yRLBFZwmZZWV1Yenb/3i/DP38yu2NxukHgk/fbo5vn3zlJ6IjIdkZVSb2u373XWF2dmxwvDQP8WfD3vnHOgyDoe77jDKyO3+/HCAHDUKo1vVTSi0VNVVWE0K/mHyUqfH/CYKtlAAAAAElFTkSuQmCC"
    },
    {
      "Name":"Ipsen,Jules U.",
      "SSN":"297-21-2682",
      "DOB":"2009-10-20",
      "Home":{
        "Street":"4291 Main Place",
        "City":"Newton",
        "State":"DE",
        "Zip":"47093"
      },
      "Office":{
        "Street":"2019 Franklin Place",
        "City":"Newton",
        "State":"VA",
        "Zip":"92649"
      },
      "Spouse":{
        "Name":"Adam,Jane C.",
        "SSN":"970-42-3768",
        "DOB":"1980-03-04",
        "Home":{
          "Street":"1395 Maple Court",
          "City":"Boston",
          "State":"IL",
          "Zip":"80443"
        },
        "Office":{
          "Street":"38 Franklin Avenue",
          "City":"Oak Creek",
          "State":"ME",
          "Zip":"42689"
        },
        "FavoriteColors":[
          "Purple"
        ],
        "Age":"40"
      },
      "Age":"10",
      "Title":"Strategic Accounts Rep.",
      "Salary":"64695",
      "Notes":"Jules used to work at BioNet LLC. as a(n) Global Developer",
      "Picture":"iVBORw0KGgoAAAANSUhEUgAAABIAAAARCAIAAABfOGuuAAAAK3RFWHRDcmVhdGlvbiBUaW1lAFRodSA2IEphbiAyMDExIDE2OjIwOjU4IC0wNTAw73VlcAAAAAd0SU1FB9sBBhUWCKSIbF4AAAAJcEhZcwAACxIAAAsSAdLdfvwAAAAEZ0FNQQAAsY8L/GEFAAAC1klEQVR42l2TyW/TQBSHZ7MdO3HsZmubtKnUTaWlKggEhwoJOCAh9W/ljjhxAEERbWmhFaVruiXYbpw43sYzHhyo2N5tpPfNfPq9edB1XfBvCQGYEClgEEAEEIYIwv9aAPn7wAXwKLcSZidRhK9kFNRIroTKGjIVLP9N/8FiDo+9dJdefOODTlBNMFCQO19s3ZaVYtIw0fyoVpUw/geLGdiyo3fBJa+2GNZVovth3YnmfbaLzA/TeP3k+mKRPWoW6wSjG4yn4ND1Xtq7tGLN5+vjuAkU7ZMn7Qjk0LGDRJspxGp4+LFNVbw2ppcz2SHaC5P19n6PbFVMboDKglRZyuWbCtQIZ0hcc8VLSQ4ODu3XO1dfKGcZgrLcOp6353x1qR0kkZs6nTi2YuhSGFMEOIQCHThnb1utIOltnzh9f4gRxkXb7bcHXUuGzrlSkUunsiIhdhYC2xcRSFJGnU4TWflxbS/wW5YbVoo5wpjo9lLnfMrrjTJzqquVLxSIcUzTLKeEoYhxRbNLNWiDVFjWteP6YtIkAAIey0lXSXxCYxGP+EH2WBZXKlKW5KTew5nO4yVvAtq9LnmR2Qs6lCQYFBVZiiQQXwmaJoMRkJfBEEsh9RpTGyuFzhiEqQ8vLS5xtVRQsySJRFC9pEwaoaG+DXjx8HQp4qbASHCOmQvI/h6xNlxRkYTncQ1NVkZ0COFwbhM19eGcfjQIpMJJaF8ffZ7hQR6wNGXRaUd2jXzk4ca01xgtP19ZNgq5m3EbuvRgYfb0zaIVOM2x47DrXu7WWNvIPL3vWiJrmo5E1Vy4s3z31pws4xsMITjbrK7Rp6822Ym/XTZZqHKey0yRLBFZwmZZWV1Yenb/3i/DP38yu2NxukHgk/fbo5vn3zlJ6IjIdkZVSb2u373XWF2dmxwvDQP8WfD3vnHOgyDoe77jDKyO3+/HCAHDUKo1vVTSi0VNVVWE0K/mHyUqfH/CYKtlAAAAAElFTkSuQmCC"
    },
    {
      "Name":"Uhles,Clint S.",
      "SSN":"469-62-1638",
      "DOB":"1944-03-26",
      "Home":{
        "Street":"3155 Elm Blvd",
        "City":"Reston",
        "State":"SC",
        "Zip":"83447"
      },
      "Office":{
        "Street":"7476 Clinton Blvd",
        "City":"Newton",
        "State":"MS",
        "Zip":"82343"
      },
      "Spouse":{
        "Name":"Paladino,Imelda S.",
        "SSN":"355-51-9101",
        "DOB":"2016-04-09",
        "Home":{
          "Street":"6070 Main Place",
          "City":"Fargo",
          "State":"ID",
          "Zip":"36015"
        },
        "Office":{
          "Street":"4662 Second Drive",
          "City":"Albany",
          "State":"VT",
          "Zip":"41903"
        },
        "FavoriteColors":[
          "Green",
          "Green"
        ],
        "Age":"4"
      },
      "FavoriteColors":[
        "Yellow"
      ],
      "Age":"76",
      "Title":"Senior Marketing Manager",
      "Salary":"488"
    },
    {
      "Name":"Evans,Zoe Y.",
      "SSN":"836-46-4116",
      "DOB":"1961-11-12",
      "Home":{
        "Street":"5158 Second Street",
        "City":"Newton",
        "State":"CO",
        "Zip":"36004"
      },
      "Office":{
        "Street":"5665 Main Avenue",
        "City":"Tampa",
        "State":"CA",
        "Zip":"22666"
      },
      "Spouse":{
        "Name":"Chadwick,David D.",
        "SSN":"867-20-2567",
        "DOB":"1982-09-04",
        "Home":{
          "Street":"1252 Franklin Avenue",
          "City":"Zanesville",
          "State":"LA",
          "Zip":"74755"
        },
        "Office":{
          "Street":"9640 Washington Court",
          "City":"Boston",
          "State":"AZ",
          "Zip":"61846"
        },
        "Age":"37"
      },
      "Age":"58",
      "Title":"Assistant Systems Engineer",
      "Salary":"58285"
    },
    {
      "Name":"Yancik,Roberta U.",
      "SSN":"864-70-8242",
      "DOB":"1998-01-12",
      "Home":{
        "Street":"9652 Maple Court",
        "City":"Chicago",
        "State":"MS",
        "Zip":"67796"
      },
      "Office":{
        "Street":"6014 Washington Court",
        "City":"Ukiah",
        "State":"WV",
        "Zip":"56011"
      },
      "Spouse":{
        "Name":"Eagleman,Valery P.",
        "SSN":"231-81-6984",
        "DOB":"1996-12-25",
        "Home":{
          "Street":"3118 Ash Street",
          "City":"Miami",
          "State":"MI",
          "Zip":"84970"
        },
        "Office":{
          "Street":"3776 Clinton Blvd",
          "City":"Ukiah",
          "State":"CT",
          "Zip":"57613"
        },
        "FavoriteColors":[
          "Green"
        ],
        "Age":"23"
      },
      "Age":"22",
      "Title":"Resources Director",
      "Salary":"75331"
    },
    {
      "Name":"Braam,Kim D.",
      "SSN":"119-77-3119",
      "DOB":"1954-01-17",
      "Home":{
        "Street":"5808 Franklin Drive",
        "City":"Elmhurst",
        "State":"MD",
        "Zip":"19151"
      },
      "Office":{
        "Street":"4002 Oak Street",
        "City":"Xavier",
        "State":"NH",
        "Zip":"90182"
      },
      "Spouse":{
        "Name":"Ulman,Ralph Z.",
        "SSN":"686-57-5448",
        "DOB":"2014-07-24",
        "Home":{
          "Street":"3482 Franklin Court",
          "City":"Bensonhurst",
          "State":"IL",
          "Zip":"67956"
        },
        "Office":{
          "Street":"7438 Elm Street",
          "City":"St Louis",
          "State":"VT",
          "Zip":"51597"
        },
        "FavoriteColors":[
          "Blue",
          "Black"
        ],
        "Age":"5"
      },
      "FavoriteColors":[
        "Orange"
      ],
      "Age":"66",
      "Title":"Laboratory Hygienist",
      "Salary":"65936"
    }
  ]
}
SAMPLES>
~~~
### note ###  
This works of course also in IRIS.  

