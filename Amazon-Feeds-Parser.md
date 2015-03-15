# Amazon Feeds Parser

The inspiration for this project was a need to parse big Amazon feeds XML files (dozens of GB) in an acceptable time.

After giving up with XSLT I decided to give "xgawk" a try.
       
## XGAWK Installation

```
wget http://home.vr-web.de/Juergen.Kahrs/gawk/XML/xgawk-3.1.6a-20090408.tar.gz
gzip -d xgawk-3.1.6a-20090408.tar.gz
tar xvf xgawk-3.1.6a-20090408.tar
cd xgawk-3.1.6a-20090408
./configure
make
su -c 'make install'
```

If you intend to run xgawk to process files with more than 2^31 bytes in 32 bits architecture then take a look at http://sourceforge.net/mailarchive/forum.php?thread_name=dff43e6d1002131449k438c91acj7458637126595e08%40mail.gmail.com&forum_name=xmlgawk-users

If you run into problems take a loot at the "Troubleshooting" section below.

## The project

The project is hosted at http://nestorurquiza.googlecode.com/svn/trunk/amazon-feeds/
This is an early phase and by the time of this writing it is composed of three xgawk files, one perl script to manage processing of multiple feeds concurrently and three folders: tmp, output and xml-sources. I have provided some sample chunks (from the big Amazon feeds like for example the biggest one "Apparel" root Category) below xml-sources.
```
.
|-- amz_categories.xgawk
|-- amz_product_category.xgawk
|-- amz_products.xgawk
|-- amz_xml_to_tsv.pl
|-- tmp
|-- output
`-- xml-sources
    |-- categories
    |-- chunk_us_ecs_apparel.xml
    |-- chunk_us_ecs_cellphone.xml
    |-- chunk_us_ecs_watches.xml
    |-- product_category
    `-- products
```

To process individual feeds use:
```
xgawk -f amz_categories.xgawk us_ecs_apparel.xml > amz_categories
xgawk -f amz_products.xgawk us_ecs_apparel.xml > amz_products
xgawk -f amz_product_category.xgawk us_ecs_apparel.xml > amz_product_category
```
You will end up with three tab separated values (tsv) files that should be enough to start rendering and searching Amazon products and nodes (or categories).

Of course you need to have xgawk ready. Below are the steps I followed in my local MacBook Pro (2.26 GHz Intel Core 2 Duo, 4GB 1067 MHz DDR3)

 1. Download the sources from the xgawk official site.
 1. Uncompress them. (in my case xgawk-3.1.6a-20090408)
 1. Run the below commands:
```
cd xgawk-3.1.6a-20090408/
./configure
make
sudo make install
```
 1. Add the below line to file ~/.bash_profile
```
export PATH=/opt/local/bin:/opt/local/sbin:$PATH
```

To process several feeds use the perl script with no parameters. Note you need to change the $PROJECT_HOME and $GZIP_SOURCES_FOLDER variables from inside the script to point to your local project base directory and the directory where the already downloaded feeds are.

After you run it you will get some data below the output directory. All ".tsv" files are created directly from each root category feed xml file. Note that the tmp directory is needed for the script to run.
```
-- output
|   |-- chunk_us_ecs_apparel_categories.tsv
|   |-- chunk_us_ecs_apparel_product_category.tsv
|   |-- chunk_us_ecs_apparel_products.tsv
|   |-- chunk_us_ecs_cellphone_categories.tsv
|   |-- chunk_us_ecs_cellphone_product_category.tsv
|   |-- chunk_us_ecs_cellphone_products.tsv
|   |-- chunk_us_ecs_watches_categories.tsv
|   |-- chunk_us_ecs_watches_product_category.tsv
|   `-- chunk_us_ecs_watches_products.tsv
```

Even though the included fields are a subset of all available you can edit the corresponding xgawk file to include more or delete some of them. I have provided some filtering which you can change as well for example I filter just Amazon Merchant (ATVPDKIKX0DER) Products. For a complete list of changes (many of them customizations) check http://code.google.com/p/nestorurquiza/source/list

Now you can import the files in your local DB Engine.

## Optimizing Resources

Amazon provides a Perl script to download just the feeds that have changed. On top of that I create symlinks for the new feeds and run the Amazon Feeds Parser just for those.

## Performance

Below are some numbers from my local environment. I have to say that by the time the scripts were running I was writing emails, browsing the web, programming in Perl, awk and Java, even chatting on skype and msn. I did not experience any big delays on any of those tasks but of course they might have affected the below results. I might come back and post some results from our Red Hat Servers when the package is deployed there.

The file parsed here (us_ecs_apparel.xml) is 34GB. I got:
```
       72min for 175033 products (id, title, price, description) resulting in 20MB of tsv file.
       74min for 7866 nodes/categories (id, parent, name) resulting in 223KB of tsv file.
       74min for 965733 product-nodes/product_categories (product_id, node_id) mappings resulting in 19MB of tsv file.
```

After I added image, height and width fields for small, medium and large images (total 9 fields) the processing time increased considerably:
```
       140min for 175033 products (id, title, price, description, lg_img, md_img, sm_img, lg_height, md_height, sm_height, lg_width, md_width, sm_width) resulting in 75MB of tsv file.
```

You can get the total execution time per script while commenting out the below line:
```
#print "Took me " systime()-start_time " seconds"
```

Memory footprint was very slow (not even 2MB of physical and half GB virtual memory). they were processor intensive tasks of course.

The more challenging code to write was the script amz_categories.xgawk as combinations of nodes and parents are repeated again and again per product. The technique I have used is just inserting into a two-dimensional array (which in awk language can be seen actually as a two-dimensional hash) both child and parent as keys. That is the reason why memory will be used and no output from the command will be available up to the moment it finishes running.

## Sample Amazon XML

Below is an Item node to show the XML elements from the Amazon feed. This corresponds to the same format used by the PA-API.

```
<Item>
  <ASIN>B0000EIPVJ</ASIN>
  <SmallImage xmlns:aws="http://webservices.amazon.com/AWSECommerceService">
    <aws:URL>http://ecx.images-amazon.com/images/I/41AQZE8GPGL.*SL75*.jpg</aws:URL>
    <aws:Height Units="pixels">75</aws:Height>
    <aws:Width Units="pixels">75</aws:Width>
  </SmallImage>
  <MediumImage xmlns:aws="http://webservices.amazon.com/AWSECommerceService">
    <aws:URL>http://ecx.images-amazon.com/images/I/41AQZE8GPGL.*SL160*.jpg</aws:URL>
    <aws:Height Units="pixels">160</aws:Height>
    <aws:Width Units="pixels">160</aws:Width>
  </MediumImage>
  <LargeImage xmlns:aws="http://webservices.amazon.com/AWSECommerceService">
    <aws:URL>http://ecx.images-amazon.com/images/I/41AQZE8GPGL.jpg</aws:URL>
    <aws:Height Units="pixels">225</aws:Height>
    <aws:Width Units="pixels">225</aws:Width>
  </LargeImage>
  <ItemAttributes>
    <Binding>Apparel</Binding>
    <Brand>ChoiceShirts</Brand>
    <Department>mens</Department>
    <FabricType>cotton-blend</FabricType>
    <Label>CHOICESHIRTS</Label>
    <Manufacturer>CHOICESHIRTS</Manufacturer>
    <ProductGroup>Apparel</ProductGroup>
    <ProductTypeName>SWEATER</ProductTypeName>
    <Publisher>CHOICESHIRTS</Publisher>
    <SKU>9110807-0</SKU>
    <Studio>CHOICESHIRTS</Studio>
    <Title>Two Dragons Adult Sweatshirt</Title>
  </ItemAttributes>
  <EditorialReviews>
    <EditorialReview>
      <Source>Product Description</Source>
      <Content>Premium-weight, high quality garment made of 50% cotton and 50% polyester with design printed on the front.</Content>
      <IsLinkSuppressed>0</IsLinkSuppressed>
    </EditorialReview>
  </EditorialReviews>
  <BrowseNodes>
    <BrowseNode>
      <BrowseNodeId>388289011</BrowseNodeId>
      <Name>Sweatshirts</Name>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>388287011</BrowseNodeId>
          <Name>Novelty</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>2227030011</BrowseNodeId>
              <Name>Specialty Apparel</Name>
              <Ancestors>
                <BrowseNode>
                  <BrowseNodeId>1036682</BrowseNodeId>
                  <Name>Departments</Name>
                  <Ancestors>
                    <BrowseNode>
                      <BrowseNodeId>1036592</BrowseNodeId>
                      <Name>Clothing</Name>
                    </BrowseNode>
                  </Ancestors>
                </BrowseNode>
              </Ancestors>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
    <BrowseNode>
      <BrowseNodeId>368723011</BrowseNodeId>
      <Name>Long Sleeves</Name>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>368720011</BrowseNodeId>
          <Name>Sleeve Length (feature_browse-bin)</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>1292216011</BrowseNodeId>
              <Name>Unlaunched Refinements</Name>
              <Ancestors>
                <BrowseNode>
                  <BrowseNodeId>15683091</BrowseNodeId>
                  <Name>Refinements</Name>
                  <Ancestors>
                    <BrowseNode>
                      <BrowseNodeId>1036592</BrowseNodeId>
                      <Name>Clothing</Name>
                    </BrowseNode>
                  </Ancestors>
                </BrowseNode>
              </Ancestors>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
    <BrowseNode>
      <BrowseNodeId>1253093011</BrowseNodeId>
      <Name>Blends</Name>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>1253091011</BrowseNodeId>
          <Name>Sweater Material (material_browse)</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>386533011</BrowseNodeId>
              <Name>Browse Refinements</Name>
              <Ancestors>
                <BrowseNode>
                  <BrowseNodeId>15683091</BrowseNodeId>
                  <Name>Refinements</Name>
                  <Ancestors>
                    <BrowseNode>
                      <BrowseNodeId>1036592</BrowseNodeId>
                      <Name>Clothing</Name>
                    </BrowseNode>
                  </Ancestors>
                </BrowseNode>
              </Ancestors>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
    <BrowseNode>
      <BrowseNodeId>382343011</BrowseNodeId>
      <Name>Crewneck</Name>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>382340011</BrowseNodeId>
          <Name>Neck Style (feature_three_browse-bin)</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>386533011</BrowseNodeId>
              <Name>Browse Refinements</Name>
              <Ancestors>
                <BrowseNode>
                  <BrowseNodeId>15683091</BrowseNodeId>
                  <Name>Refinements</Name>
                  <Ancestors>
                    <BrowseNode>
                      <BrowseNodeId>1036592</BrowseNodeId>
                      <Name>Clothing</Name>
                    </BrowseNode>
                  </Ancestors>
                </BrowseNode>
              </Ancestors>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
    <BrowseNode>
      <BrowseNodeId>368893011</BrowseNodeId>
      <Name>Blends</Name>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>368885011</BrowseNodeId>
          <Name>Material (material_browse)</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>386533011</BrowseNodeId>
              <Name>Browse Refinements</Name>
              <Ancestors>
                <BrowseNode>
                  <BrowseNodeId>15683091</BrowseNodeId>
                  <Name>Refinements</Name>
                  <Ancestors>
                    <BrowseNode>
                      <BrowseNodeId>1036592</BrowseNodeId>
                      <Name>Clothing</Name>
                    </BrowseNode>
                  </Ancestors>
                </BrowseNode>
              </Ancestors>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
    <BrowseNode>
      <BrowseNodeId>251277011</BrowseNodeId>
      <Name>Custom Stores</Name>
      <Children>
        <BrowseNode>
          <BrowseNodeId>1294227011</BrowseNodeId>
          <Name>Brand Stores</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2226736011</BrowseNodeId>
          <Name>Athletic Apparel Boutique</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2224749011</BrowseNodeId>
          <Name>Cleanup Brand Stores</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1232947011</BrowseNodeId>
          <Name>Cold Weather Apparel</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2210867011</BrowseNodeId>
          <Name>Denim Shop</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>13952391</BrowseNodeId>
          <Name>Designer Specialty Store</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2224935011</BrowseNodeId>
          <Name>Fall Fashion TEST</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1258977011</BrowseNodeId>
          <Name>Maternity</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2225177011</BrowseNodeId>
          <Name>Men's Fall Fashion</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1297150011</BrowseNodeId>
          <Name>Outdoor &amp; Camping</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1297113011</BrowseNodeId>
          <Name>Outerwear</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1263416011</BrowseNodeId>
          <Name>Plus Size</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>689283011</BrowseNodeId>
          <Name>School Uniforms</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>1260917011</BrowseNodeId>
          <Name>Test Platinum &amp; Gold Generalization</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>689282011</BrowseNodeId>
          <Name>Wedding</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2055486011</BrowseNodeId>
          <Name>What to Wear</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2225176011</BrowseNodeId>
          <Name>Women's Fall Fashion</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>689281011</BrowseNodeId>
          <Name>Work Apparel &amp; Uniforms</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2227013011</BrowseNodeId>
          <Name>Quicklist Popover Content</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2229956011</BrowseNodeId>
          <Name>Men's Fall Trends</Name>
        </BrowseNode>
        <BrowseNode>
          <BrowseNodeId>2229957011</BrowseNodeId>
          <Name>Women's Fall Trends</Name>
        </BrowseNode>
      </Children>
      <Ancestors>
        <BrowseNode>
          <BrowseNodeId>1036684</BrowseNodeId>
          <Name>Specialty Stores</Name>
          <Ancestors>
            <BrowseNode>
              <BrowseNodeId>1036592</BrowseNodeId>
              <Name>Clothing</Name>
            </BrowseNode>
          </Ancestors>
        </BrowseNode>
      </Ancestors>
    </BrowseNode>
  </BrowseNodes>
<PriceSummary>
  <hasVariations/>
  <ItemPrice>
    <Amount>2195</Amount>
    <Availability>In Stock</Availability>
    <AvailabilityMinHours>0</AvailabilityMinHours>
    <AvailabilityMaxHours>0</AvailabilityMaxHours>
    <SalesRestriction/>
    <MerchantID>A2E3MC4E0XDV6S</MerchantID>
    <ShippingCharge>
      <ShippingPrice>
        <FormattedPrice>Check Site.</FormattedPrice>
      </ShippingPrice>
    </ShippingCharge>
  </ItemPrice>
  <MPPrice type="">
    <Amount>2195</Amount>

    <Availability>In Stock</Availability>
    <AvailabilityMinHours>0</AvailabilityMinHours>
    <AvailabilityMaxHours>0</AvailabilityMaxHours>
    <SalesRestriction/>
    <MerchantID>A2E3MC4E0XDV6S</MerchantID>
    <ShippingCharge>
      <ShippingPrice>
        <FormattedPrice>Check Site.</FormattedPrice>
      </ShippingPrice>
    </ShippingCharge>
  </MPPrice>
  <ThirdPartyNewPrice>
    <Amount>2195</Amount>
    <Availability>In Stock</Availability>
    <AvailabilityMinHours>0</AvailabilityMinHours>
    <AvailabilityMaxHours>0</AvailabilityMaxHours>
    <SalesRestriction/>
    <MerchantID>A2E3MC4E0XDV6S</MerchantID>
    <ShippingCharge>
      <ShippingPrice>
        <FormattedPrice>Check Site.</FormattedPrice>
      </ShippingPrice>
    </ShippingCharge>
  </ThirdPartyNewPrice>
</PriceSummary>
<Description>
  <Abstract>Buy Two Dragons Adult Sweatshirt from Amazon.com!</Abstract>
  <Promotion>Get free shipping on orders over $25!</Promotion>
  <AmazonUrl>http://www.amazon.com/dp/B0000EIPVJ/ref=asc_df_B0000EIPVJ947164/?tag=INSERT_TAG_HERE&amp;creative=380333&amp;creativeASIN=B0000EIPVJ&amp;linkCode=asn</AmazonUrl>
  <MarketplaceURL>http://www.amazon.com/gp/product/B0000EIPVJ/ref=asc_df_B0000EIPVJ947164?s=merchant&amp;m=&amp;v=glance&amp;tag=INSERT_TAG_HERE&amp;creative=380341&amp;linkCode=asn&amp;creativeASIN=B0000EIPVJ</MarketplaceURL>
  <OfferUrl>http://www.amazon.com/gp/offer-listing/B0000EIPVJ/ref=asc_df_B0000EIPVJ947164?ie=UTF8&amp;condition=new&amp;tag=INSERT_TAG_HERE&amp;creative=380345&amp;creativeASIN=B0000EIPVJ&amp;linkCode=asm</OfferUrl>
  <Url>http://www.amazon.com/dp/B0000EIPVJ/ref=asc_df_B0000EIPVJ947164?tag=INSERT_TAG_HERE&amp;creative=380333&amp;creativeASIN=B0000EIPVJ&amp;linkCode=asn</Url>
  <UsedUrl>http://www.amazon.com/gp/offer-listing/B0000EIPVJ/ref=asc_df_B0000EIPVJ947164?ie=UTF8&amp;condition=used&amp;tag=INSERT_TAG_HERE&amp;creative=380349&amp;creativeASIN=B0000EIPVJ&amp;linkCode=asm</UsedUrl>
</Description>
</Item>
```

## Troubleshooting

At least for Ubuntu 64 bits I was getting the below message:
```
# xgawk -l xml
xgawk: fatal: extension: cannot find dynamic library `xml'
```
It turned out to be a configuration problem:
```
./configure:
...
checking expat.h usability... no
checking expat.h presence... no
checking for expat.h... no
...
```
So to install expat in Ubuntu just run the below and reinstall xgawk:
```
#apt-get install expat
#apt-get install libexpat1-dev
```

You can always get help while subscribing to xgawk mailing list (https://lists.sourceforge.net/lists/listinfo/xmlgawk-users) or perhaps browsing older posts from http://sourceforge.net/mailarchive/forum.php?forum_name=xmlgawk-users

## Contributing

Please report any bugs or contributions to nestor dot urquiza at gmail dot com.