---
title: MongoDB Issues and Fixes
---

### Topics
- DotNet
  - Driver doesn't convert empty String values to default value of primitive type. Example if a field type is `int` and field value in data comes blank "" it will throw `BadFormatException`
  - Workaround
  	- Create a Null check serializer for that type for which you think you might receive dirty data
    ```csharp
    public class NullIntSerializer : IBsonSerializer<int> {
  	  public override int Deserialize(BsonDeserializationContext context, BsonDeserializationArgs args) {
  		  // No Change when BsonType matched target type
  		  if (context.Reader.CurrentBsonType == BsonType.Int32)
  			  return context.Reader.ReadInt32();
  		  
  		  // if current BsonType is string and empty provide default value for target
  		  if (context.Reader.CurrentBsonType == BsonType.String) {
  			  var val = context.Reader.ReadString();
  			  return int.TryParse(val, out int intVal) ? intVal : 0;
  		  }
  	  }
   
  	  public override void Serialize(BsonSerializationContext context, BsonSerializationArgs args, int value) {
  		  var bsonWriter = context.Writer;
  		  if (value == null || string.IsNullOrWhiteSpace(value.ToString())) {
  			  bsonWriter.WriteNull();
  			  return;
  		  }
  		  else {
  			  bsonWriter.WriteInt32(value);
  		  }
  	  }
    }
    ```
	- Assign Serializer to the Field
	  ```csharp
	  BsonClassMap.RegisterClassMap<Movie>(x => {
			x.AutoMap(); // It is necessary as you are overriding default behavior and all mappings are reset
			x.MapMember(x => x.rating).SetSerializer(new NullIntSerializer());
		});
	  ```
  - `BsonTypes` provided by Library `MongoDB.Bson` are not compatible with Swagger. Swagger UI crashes or hangs when they are used as Data types in models
    - Workaround
      - Use `BsonRepresentationAttribute` instead of BsonType as return types of fields
		```csharp
		[BsonRepresentation(BsonType.ObjectId)]
		public string _id { get; set; }
		```
  - If Model has field `id` as field name then driver treats it as `_id' field even if it is not meant for that purpose
	- Workaround
    	- Change the field name to something other than Id. Example `ImdbId` and provide a `BsonElementAttribute`
    	```csharp
    	[BsonElement("id")]
    	public string ImdbId { get; set; }
    	```
  - Debug Mongo Source Code for correct filter operations in class MongoCollectionImpl<TDocument> 
	
