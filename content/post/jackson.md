+++
linktitle = "Streaming JSON with Jackson"
featured = ""
featuredpath = ""
date = "2017-02-08T20:27:44+01:00"
title = "Streaming JSON with Jackson"
author = "Radomir Sohlich"
categories = [ "java"
]
description = ""
featuredalt = ""

+++

Sometimes there is a situation, when it is more 
efficient to parse the JSON in stream way. 
Especially if you are dealing with huge input 
or slow connection. In that case the JSON need to be read 
as it comes from input part by part.
The side effect of such approach is that you are able to
read corrupted JSON arrays in some kind of comfortable way.

Well known Jackson library (https://github.com/FasterXML/jackson-core) 
provides so called stream API to handle it. 
The parsing of stream is possible via 
JsonParser object, which uses token approach.

```
InputStream is = new FileInputStream("data.json");
JsonFactory factory = new JsonFactory();
JsonParser parser = factory.createJsonParser(is);
```

To simplify the approach, the combination of ObjectMapper 
with sequetial reading of JSON data could be used.The code of the 
complete solution could be like:
```
public void tryParse(InputStream is) throws IOException {
		JsonFactory factory = new JsonFactory();
		JsonParser parser = factory.createJsonParser(is);
		ObjectMapper mapper = new ObjectMapper()
				.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES,false);

		JsonToken token = parser.nextToken();
		
        	// Try find at least one object or array.
		while (!JsonToken.START_ARRAY.equals(token) && token != null && !JsonToken.START_OBJECT.equals(token)) {
			parser.nextToken();
		}

		// No content found
		if (token == null) {
			return;
		}

		boolean scanMore = false;

		while (true) {
			// If the first token is the start of obejct ->
			// the response contains only one object (no array)
			// do not try to get the first object from array.
			try {
				if (!JsonToken.START_OBJECT.equals(token) || scanMore) {
					token = parser.nextToken();
				}
				if (!JsonToken.START_OBJECT.equals(token)) {
					break;
				}
				if (token == null) {
					break;
				}

				Object node = mapper.readValue(parser, mappedClass);
				
				//YOUR CODE TO HANDLE OBJECT
				//...


				scanMore = true;
			} catch (JsonParseException e) {
				handleParseException(e);
				break;
			}
		}
	}
```



