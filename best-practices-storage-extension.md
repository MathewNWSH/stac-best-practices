# Storage Extension Best Practices

## Overview of the extension

The Storage Extension is a powerful set of attributes that provides information about the method and environment used to store data whose paths are exposed in a STAC catalog. Its proper implementation enriches href paths with additional context, delivering information about the cloud environment and its characteristics where the actual data is stored. This directly translates to improved user experience, as users gain clarity about their data's location, the service provider being used and the appropriate endpoint to access.

## Provided Fields

### `storage:schemas`

The extension allows for custom definition of the object storage being used. This means there isn't a single `storage` attribute as an enum with values corresponding to the most popular storage providers. Instead, you independently populate the provided fields to characterize your storage as precisely as possible. **This is accomplished through the `storage:schemas` key implemented at the item's properties level**, whose value consists of nested JSONs. The keys of these JSONs are the names of your schemas (e.g., `s3_storage`), while the values are sets representing their characteristics. The available fields include:

- **type** (required) - The provider identifier. Currently, three types are supported: `ms-azure`, `aws-s3`, and `custom-s3`. If your data is stored on one of the first two providers listed, the choice is straightforward—use one of those. However, if you're using providers such as Wasabi, CREODIAS, Alibaba Cloud, or a self-hosted S3 protocol, you'll need to use `custom-s3`, which serves as a placeholder for less common providers.

- **platform** (required) - A URI representation of an endpoint or endpoint schema where the data is stored.

- **region** - The region of the cloud infrastructure. Ideally, if you provide multiple locations, each should be represented in a different schema so users can choose the physically closest one for the fastest data access.

- **requester_pays** - Informs users whether data access is billed or provided free of charge.

- **Any other relevant information** - While the two fields listed above are mandatory and two are optional, the extension's author has given users the flexibility to expand the standard set according to their own needs. For example, if you'd like to include the bucket name, simply add a `bucket` attribute with its string value. It's that simple. For additional examples like this, consult the documentation for [AWS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html), [CDSE](https://documentation.dataspace.copernicus.eu/APIs/S3.html), or [CREODIAS](https://creodias.docs.cloudferro.com/en/latest/s3/Configuration-files-for-s3cmd-command-on-Creodias.html) and map any attributes you find relevant to your use case.

Example of an `custom-s3` type of schema used by [CDSE STAC Catalogue](https://browser.stac.dataspace.copernicus.eu/?.language=en) can be found below:

```json
  ...
  "properties": {
    ...
    "storage:schemes": {
      "cdse-s3": {
        "type": "custom-s3",
        "title": "Copernicus Data Space Ecosystem S3",
        "platform": "https://eodata.dataspace.copernicus.eu",
        "description": "This endpoint provides access to EO data which is stored on the object storage of both CloudFerro Cloud and OpenTelekom Cloud (OTC). See the [documentation](https://documentation.dataspace.copernicus.eu/APIs/S3.html) for more information, including how to get credentials.",
        "requester_pays": false
      },
      "creodias-s3": {
        "type": "custom-s3",
        "title": "CREODIAS S3",
        "platform": "https://eodata.cloudferro.com",
        "description": "Comprehensive Earth Observation Data (EODATA) archive offered by CREODIAS as a commercial part of CDSE, designed to provide users with access to a vast repository of satellite data without predefined quota limits.",
        "requester_pays": true
      }
    },
    ...
```

Here, two types of schemas are defined: the public one through the CDSE initiative and the commercial one via CREODIAS, representing the commercial aspect of CDSE. This is an excellent example of how helpful the Storage Extension is and how it enables differentiation between free and billed connections. Additionally, both schemas have been enriched with an additional `description` attribute, which isn't included in the extension's standard set. However, thanks to the extension's flexibility, it can be added to better emphasize the differences between the two options.

Example of an `aws-s3` connection with greater emphasis on URI-like platform definition:

```json
  ...
  "properties": {
    ...
    "storage:schemas": {
      "aws-std": {
        "type": "aws-s3",
        "platform": "https://{bucket}.s3.{region}.amazonaws.com",
        "bucket": "naip-visualization",
        "region": "us-west-2",
        "requester_pays": true
      }
    },
    ...
  }
  ...
```

This example demonstrates how the `platform` attribute can be defined as a URI template with placeholders (`{bucket}` and `{region}`), which are then resolved using the values provided in the `bucket` (another additional field) and `region` fields. Both examples are valid.

`storage:schemas` can be defined at three levels:

- Catalog
- Collection
- Item's Properties

### `storage:refs`

Jest to atrybut imlementowany na poziomie Assetów, zarówno kolekcji jak i itemów, czyli realnych fiznycznych plików lub danych, do których prowadzi katalog. `storage:refs` to lista stringów, a możliwe elementy tej listy to wcześniej zdefiniowane klucze w `storage:schemas`.

Poniżej przykład w jaki sposob zakmonunikowac uzytkownikowi ze asset jest dostepny na obu ze zdefiniowanych schemas:

```json
    "some_asset": { #asset level implementation
      ...
      "href": "s3://...",
      "storage:refs": [
        "cdse-s3",
        "creodias-s3"
      ],
      ...
    },
```

W przypadku, gdy wybrany asset jest dostępny tylko u jednego providera:

```json
    "some_other_asset": { #asset level implementation
      ...
      "href": "s3://...",
      "storage:refs": [
        "aws-s3"
      ],
      ...
    },
```

Oraz ważny przypadek, **gdy wśród assetów są też assety, które wydawane są przez API, nie zapewniono dostępu via s3**:

```json
    "yet_another_asset": { #asset level implementation
      ...
      "href": "https://...",
      "storage:refs": [
      ],
      ...
    },
```

However, its primary purpose is to describe s3 access that truly matters. For example, it's recommended to consider whether S3 access needs to be highlighted for public bucket URLs pointing to assets like manuals or other PDF files. There's a low likelihood that users will want to access public documentation data via the S3 standard.

## Value added by combination with other extensions

### Alternate Assets Extension

### Authentication Extension
