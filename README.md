# Thumbnails Convention Metadata

- **UUID**: 38a1d2ca-5f40-4ee2-b4d5-5e87bfeb7549
- **Name**: Thumbnails
- **Schema URL**: "https://raw.githubusercontent.com/zarr-conventions/thumbnails/refs/tags/v2/schema.json"
- **Spec URL**: "https://github.com/zarr-conventions/thumbnails/blob/v2/README.md"
- **Scope**: Array, Group
- **Extension Maturity Classification**: Proposal
- **Owner**: @clbarnes

## Description

This convention allows a Zarr node to refer to thumnbails which represent that node in some way.
All properties are under the `thumbnails` key placed at the root `attributes` level following the [Zarr Conventions Specification](https://github.com/zarr-conventions/zarr-conventions-spec).

Thumbnails SHOULD be stored in a well-supported image format (e.g. JPEG, PNG).

Thumbnails SHOULD include at least one small image (under 200KB and longest edge between 128px and 512 px) suitable for fast loading in an image gallery.
Thumbnails MAY include larger files for detail panels or hover previews.
Thumbnails MAY include smaller files for specialized used cases, such as icon-sized previews.

How a thumbnail "represents" a Zarr node is up to the data owner.
As examples, a thumbnail MAY be

- a low-resolution 2D cropped slice from the middle of a Zarr array
- a descriptive excerpt from one scale level of a multiscale array pyramid
- a logo

Examples:

- [Simple example also shown below](examples/spec_example.json)

## Motivation

- **Viewer consistency**: Users interacting with the same data across multiple viewers may be helped by the same thumbnails
- **Owner control**: Data owners are more likely than viewers to know which parts of an array or group is "representative"

## Convention Registration

The convention must be registered in `zarr_conventions`:

```json
{
  "zarr_conventions": [
    {
      "schema_url": "https://raw.githubusercontent.com/zarr-conventions/thumbnails/refs/tags/v2/schema.json",
      "spec_url": "https://github.com/zarr-conventions/thumbnails/blob/v2/README.md",
      "uuid": "38a1d2ca-5f40-4ee2-b4d5-5e87bfeb7549",
      "name": "thumbnails",
      "description": "Metadata for thumbnails representing Zarr data"
    }
  ]
}
```

## Applicable To

This convention can be used with these parts of the Zarr hierarchy:

- [x] Group
- [x] Array

## Properties

All properties are placed at the root `attributes` level. To avoid attribute name collisions with other conventions, this convention supports the following pattern [remove the pattern irrelevant for this convention]:

All convention properties are nested under a single `thumbnails` key.

**Convention metadata name**: `thumbnails`

| Field Name | Type                                            | Description                      |
| ---------- | ----------------------------------------------- | -------------------------------- |
| thumbnails | array of [thumbnails Object](#thumbnail-object) | **REQUIRED**. Thumbnail details. |

**Example**:

```jsonc
{
  "zarr_format": 3,
  // ...
  "attributes": {
    "zarr_conventions": [{ "name": "thumbnails", "spec_url": "..." }],
    "thumbnails": [
      {
        "width": 96,
        "height": 96,
        // optional
        "attributes": {
          // unstructured
          "z_slice": 123,
          "xy_offset": [128, 256],
          "downsampling": "gaussian",
        },
        "media_type": "image/jpeg",
        // relative path downward from this zarr node to the thumbnail storage key
        "uri": "thumbails/thumb96.jpeg",
      },
      {
        "width": 48,
        "height": 48,
        "media_type": "image/png",
        // optional
        "description": "Very small thumbnail",
        // URL to external
        "uri": "https://image.host/thumb48.png",
      },
    ],
  },
}
```

### Thumbnail Object

When using the nested pattern, all properties are contained in the `thumbnails` object:

| Field Name  | Type   | Description                                                                                                                                            |
| ----------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| width       | number | **REQUIRED**. Image width in pixels as a positive integer.                                                                                             |
| height      | number | **REQUIRED**. Image height in pixels as a positive integer.                                                                                            |
| media_type  | string | **REQUIRED**. [Media type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types) (formerly MIME type).                                  |
| uri         | string | **REQUIRED**. [URI-reference](https://datatracker.ietf.org/doc/html/rfc3986) to the thumbnail image.                                                   |
| description | string | Free-text description of this thumbnail's context; could be used as [alt text](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/alt). |
| attributes  | object | Unstructured arbitrary metadata about the thumbnail; could store information about how it was generated or how it represents the zarr node.            |

The `uri` field MUST contain one of

- a relative reference from the Zarr node prefix to an object on the same Zarr storage
  - this SHOULD NOT contain `..` path segments, as ascending paths may be handled differently depending on how the Zarr store was opened
- a [data URI](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Schemes/data) encoding the thumbnail data inline
  - it is RECOMENDED that only very small thumnbails are encoded in this form
- an absolute IRI with the HTTPS scheme
  - the image data MUST be available to an HTTP GET request
  - this form is NOT RECOMMENDED

## Known Implementations

This section helps potential implementers assess the convention's maturity and adoption, and provides a way for the community to collaborate on future revisions.

### Libraries and Tools

- **[zarrs_conventions_thumbnails](https://github.com/clbarnes/zarrs_conventions/tree/main/zarrs_conventions_thumbnails)** - Implementation for the rust/ zarrs/ zarrs_conventions ecosystem
  - Language: Rust
  - Status: Experimental
  - Maintainer: @clbarnes
  - Since: Version 0.1.0

### Datasets Using This Convention

- **[Example ome-ngff-validator](https://ome.github.io/ome-ngff-validator/?source=https://pub-c4a3c62df94840688feeada706020240.r2.dev/4ffaeed2-fa70-4907-820f-8a96ef683095.zarr)** - HeLa cells showing click chemistry and immunofluorescence

### Resources

- Initial proposal: <https://github.com/German-BioImaging/ome-zarr-ideas/issues/43>

_If you implement or use this convention, please add your implementation to this list by submitting a pull request._

## Acknowledgements

This thumbnails is based on the [STAC extensions thumbnails](https://github.com/stac-extensions/thumbnails/blob/main/README.md).
