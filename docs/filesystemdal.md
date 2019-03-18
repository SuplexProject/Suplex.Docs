# FileSystemDal Overview

The FileSystemDal is a simple wrapper on the Suplex.Security.DataAccess.MemoryDal, which is an in-memory Suplex store.  The FileSystemDal adds serialize/deserialize methods to support persistence.

## Key Properties

|Field/Method|Type|Required|Description
|-|-|-|-
|CurrentPath|string|Yes|Path for serialize/deserialize actions.
|AutomaticallyPersistChanges|bool|Yes|Indicates whether to automatically serialize the store to the CurrentPath when changes occur. Default is **false**.
|SerializeAsJson|bool|Yes|Indicates whether the store will serialize as JSON or YAML (default). Default is **false**.

## Key Methods

|Type|Method|Return Value|Description
|-|-|-|-
|Instance|ToYaml(bool serializeAsJson = false)|string|Serializes the store to YAML, or optionally to JSON.
|Instance|ToYamlFile(string path = null, bool serializeAsJson = false)|void|Serializes the store to YAML, or optionally to JSON, and persists the result to a file.  If `path` is not provided, `CurrentPath` is assumed.  If `path` is provided, it will override and reset `CurrentPath`.
|Instance|FromYaml(string yaml)|void|Deserializes a YAML or JSON string blob and initializes a new store.
|Static|LoadFromYaml(string yaml)|FileSystemDal|Deserializes a YAML or JSON string blob and initializes a new store.
|Instance|FromYamlFile(string path)|void|Deserializes a YAML or JSON file and initializes a new store.
|Static|LoadFromYamlFile(string path)|FileSystemDal|Deserializes a YAML or JSON file and initializes a new store.

## Rationale for a File Store

Security configuration data tends to be very slow moving, by nature.  Often times there is a flurry of setup activity followed by long periods of stability.  If you don't otherwise require User/Group information to be in your application database, a simple strategy for Suplex integration is to use the FileSystemDal coupled with a FileSystemWatcher, where edits to the file are handled externally from the WPF UI.  See the SampleApp for an example of this approach.

If your store experiences a high volume of changes or you otherwise require access to SecurityPrincipal data for joins/etc., consider a database DAL implementation, such as the DynamoDbDal or MongoDbDal.  