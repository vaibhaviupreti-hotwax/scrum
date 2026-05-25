# previously
- User Login data model
# Product data model

## ProdCatalog
- most popular
- best selling

## ProdCatalogCategory
- most popular menswear
- most popular kidswear
- best selling phones

## ProductCategory
- menswear
- kidswear

## ProductCategoryMember(ASSOC) -> PRODUCT <--> PRODUCT_CATEGORY

## ProductCategoryRollup
- ProductCategory relation with itself. 

# Assignment-1+2
- download plugin - ecommerce (below)
- make new catalog/category/ - with an new example

# https://ofbizdemotesting.hotwaxsystems.com/ecommerce/control/main
- seed data
- data files
- round gizmo
- productid-product[master entity - no FK constraint] (make own data)
 product_category_member [link with category - already existing/create new]
 ProdCatalogCategory - check catalogs [summerwear][womenswear]
 ProductCategoryRollup - [cat to cat]
 ProdCatalog

# download plugins - OFBiz setup
ofbiz.apache.org
community - src repo
https://ofbiz.apache.org/source-repositories.html
$ git clone https://gitbox.apache.org/repos/asf/ofbiz-plugins.git plugins
$ git checkout origin/release24.09
https://github.com/apache/ofbiz-plugins/tree/trunk/ecommerce

particular category - management will be done by xyz

# Assignment-3 (already done)
- apache derby - default to mysql
- entityEngine.xml - db cn
- db agnostic proj - independent of db
- jdbc connectivity
- db connectivity [build.gradle- dependency and lib]

# Assignment-4
- old assignment - to read terminologies (daily)
---

# Ashish Vijaywargiya: Important Terms/Terminologies from our day to day operations

Dear @all,

In our last meeting, I asked all of you to explore most commonly used terms/terminologies in the following areas:

Ecommerce OMS WMS ERP

Please refer following articles:
https://econsultancy.com/a-glossary-of-ecommerce-terminology/

https://omsdocs.magento.com/getting-started/terminology/

https://optimoroute.com/warehouse-terminology/

https://www.theaccessgroup.com/en-au/erp/software/order-management/the-a-z-of-order-management/

https://www.mintsoft.com/resources/blog/the-a-z-of-order-management-terms-and-definitions-you-need-to-know/

https://www.visualsouth.com/blog/erp-terminology

Please read these articles whenever you can. And there can be other articles as well, so keep searching and improve your knowledge base. Thank you.

-- https://www.hotwaxsystems.com/apache-ofbiz-tutorials/ofbiz/ofbiz-tutorials/tutorial-setup-an-ofbiz-e-commerce-store-catalog
