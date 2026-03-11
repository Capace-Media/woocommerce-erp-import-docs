# WooCommerce ERP Import System

This project documents the automated product import pipeline used for the following sites:

- `sko-dej.com`
- `shoesbags.se`

Both stores import product data from the same ERP export via FTP.

The system is built using WP All Import and cron jobs to automatically sync products, images, stock, and pricing.

## Architecture Overview

```
Pyramid ERP
     │
     │ (exports XML + images)
     ▼
FTP Server (ftpcluster.loopia.se)
     │
     ├── /cdn.capace.dk/public_html/xml/woocommerce.xml
     └── /cdn.capace.dk/public_html/img/*.jpg
     │
     ▼
WP All Import (WordPress)
     │
     ├── sko-dej.com
     └── shoesbags.se
```

### Flow

1. Pyramid ERP exports product data
2. Files are uploaded to the FTP server
3. WordPress downloads the XML via WP All Import
4. Products are created/updated in WooCommerce
5. Images are attached from the image folder

## Seasonal Product Replacement

The stores operate on a seasonal catalog model.

When a new XML feed is uploaded to the FTP server, the import process replaces the entire product catalog.

**Behavior**

During each import run:

1. The XML file is parsed.
2. Products are matched using the unique identifier Art.
3. Products present in the XML are created or updated.
4. Products not present in the XML are automatically removed from WooCommerce.
5. Images attached to removed products are also deleted.

This ensures the WooCommerce catalog always mirrors the ERP export exactly.

### **Example**

**Previous XML:**

```
Product A
Product B
Product C
```

**New XML uploaded for a new season:**

```
Product D
Product E
Product F
```

**After the import runs:**

```
Product A → deleted
Product B → deleted
Product C → deleted

Product D → created
Product E → created
Product F → created
```

The store will therefore always reflect the current seasonal product catalog provided by the ERP.

**Important**

If the XML feed is empty or incorrect, the import may remove all products from the store.

Always verify that the XML export from the ERP contains the expected products before running the import.

## FTP Server

### Example accounts configured on the server:

| User               | Access description            |
| ------------------ | ----------------------------- |
| `erp_ftp_webroot`  | Web root for XML + images     |
| `erp_ftp_xml_only` | XML directory only            |
| `erp_ftp_admin`    | Full account (administrative) |

These example accounts illustrate typical access levels used by the ERP export.  
**Actual usernames, passwords, and hosting control‑panel details should be stored only in internal, private documentation.**

## Important Notes

**Shared data source**

Both websites rely on the same FTP export.

If the XML feed changes or breaks, both sites are affected.

## Product deletion

If a product disappears from the XML feed, it will be removed from WooCommerce on the next import run.

## Independent WooCommerce stores

Even though the data source is shared:

- Products are stored separately
- Media libraries are separate
- Import configurations are separate

## Maintenance

If imports stop working, check:

1. FTP connection
2. XML file exists
3. Image files exist
4. Cron jobs are running at easycron.com (Reset execution logs and statistics can help)
5. WP All Import logs
