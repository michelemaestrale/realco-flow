# realco-flow
Realco pre tranformation to build Nexla compliant csv
REALCO pre Nexla Flow transformation Docs

GitHub link: 
Main purpose:

Operate a transformation on the file .xml daily uploaded by Realco to make it readable for Nexla, this is a temporary workaround not to delay anymore the Nexla flow start date to avoid partner bad experience and churn.

Ops

Realco upload on a daily basis at 9:00 am a file named realco_cedi_ecommerce.xml that contain all the product of the assortment
Realco uploads on a daily basis at 9:00 am a file named PDV-{store_id}.xml that contains all the available products in the store (store_id) and info about price and promo_price of the product. 

File structure:

realco_cedi_ecommerce.xml file structure:



<?xml version="1.0" encoding="UTF-8"?>
<Articoli PDV="REALCO" DescrPDV="CEDI">
    <Articolo>
        <ProductId>10001</ProductId>
        <EANs>8003185000010</EANs>
        <Description>Aceto balsamico di Modena IGP MONARI &amp; FEDERZONI vivace 50 cl</Description>
        <VendorCategory>0105010201</VendorCategory>
        <QtUmFiscale>500</QtUmFiscale>
        <UmFiscale>ml</UmFiscale>
        <QuantityToSell>0</QuantityToSell>
        <SizeToSell></SizeToSell>
        <DaPesare>false</DaPesare>
        <ProducesSingleEAN>false</ProducesSingleEAN>
        <OtherInfo></OtherInfo>
    </Articolo>
    <Articolo>
        <ProductId>10008</ProductId>
        <EANs>8004348140444</EANs>
        <Description>Aceto balsamico di Modena IGP SIGMA 50cl</Description>
        <VendorCategory>0105010201</VendorCategory>
        <QtUmFiscale>500</QtUmFiscale>
        <UmFiscale>ml</UmFiscale>
        <QuantityToSell>0</QuantityToSell>
        <SizeToSell></SizeToSell>
        <DaPesare>false</DaPesare>
        <ProducesSingleEAN>false</ProducesSingleEAN>
        <OtherInfo></OtherInfo>
    </Articolo>


This file contains all the product of the Realco assortment, 
From this file has been extracted the xls menu on admin and a key it has been used the <ProductId>10008</ProductId>  number as a string data type



Realo PDV-{store_id}.xml file structure:

<?xml version="1.0" encoding="utf-8"?>
<Articoli PDV="1111" DescrPDV="SIGMA Parma Gramsci">
    <Articolo>
        <ProductId>10001</ProductId>
        <EANs> 8003185000010</EANs>
        <Description>AC BAL MO IGP MF G VIVACE 50cl</Description>
        <IsFidelity>false</IsFidelity>
        <IsPromo>false</IsPromo>
        <PricePromo>0.00</PricePromo>
        <PromoStart/>
        <PromoEnd/>
        <Price>2.79</Price>
    </Articolo>
   <Articolo>
        <ProductId>10021</ProductId>
        <EANs> 80228486</EANs>
        <Description>ACETO BALSAMICO PONTI 6   50cl</Description>
        <IsFidelity>false</IsFidelity>
        <IsPromo>true</IsPromo>
        <PricePromo>1.99</PricePromo>
        <PromoStart>18/01/2023</PromoStart>
        <PromoEnd>29/01/2023</PromoEnd>
        <Price>3.09</Price>
    </Articolo>



Transformations:


<Articoli PDV="{store_id}" DescrPDV="SIGMA Store Description"> 

{store_id} reflect the column store_id in the final csv

    <Articolo>
        <ProductId>10001</ProductId> 
        <EANs> 8003185000010</EANs>
        <Description>AC BAL MO IGP MF G VIVACE 50cl</Description>
        <IsFidelity>false</IsFidelity>
        <IsPromo>false</IsPromo>
        <PricePromo>0.00</PricePromo>  —> reflect the price column in csv if greater than zero
        <PromoStart/>
        <PromoEnd/>
        <Price>2.79</Price>   —> reflect the price column in csv if PricePromo is equal to zero
    </Articolo>




The transformation tool is set up to be run (manually or automatically) daily after the upload of the xml file by Realco.
It create a csv with the following structure:

store_id,product_id,price,stock
1111,10001,2.79,10
1111,10008,1.69,10
1111,100129,4.39,10
1111,10016,1.69,10
1111,380630,9999,0
1111,380635,9999,0
1111,380639,9999,0
1111,380642,9999,0


The Header is standard header for Nexla reading capabilities for Glovo Flows

Store_id column, since there is a file for each xml uploaded, reflect the PDV attribute




The first n rows  are picked up from the PDV-{store_id}.xml file with the following logic:
If promo price is equal to zero 0 picks up for the column price <Price>3.09</Price> 
If promo price is greater than zero 0  picks up for the column price <PricePromo>0.00</PricePromo> : 1111,10001,2.79,10
The last m rows are built up following this logic:
All product of the menu are stored in a list and updated daily with the realco_cedi_ecommerce.xml elements
All products that are in the manu list but not in the PDV-{store_id}.xml  are appended to the csv as non available with 999 price: 1111,380630,9999,0

The transformation tool pick up the store_id an save it as a variable 
