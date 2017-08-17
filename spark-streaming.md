
## Spark Streaming
#### Read data from Kafka
```
val topicsSet = "Topic1,Topic2,Topic3".split(",").toSet
val brokerList = "kafkaBroker1:9092"
val kafkaParams = Map[String, String]("metadata.broker.list" -> brokerList)
      
val kafkaDStream: DStream[(String, Array[Byte])] = KafkaUtils.createDirectStream[String, Array[Byte], StringDecoder, DefaultDecoder](ssc, kafkaParams, topicsSet)
val rawData: DStream[Array[Byte]] = kafkaDStream.map(_._2) // Drop Keys from Kafka Key-Value 
```

#### Deserialize Avro data

```
{
  val records: DStream[CustomAvroClass] = rawData.map(x => deSerializeAvro(x))
}

def deSerializeAvro(bytes: Array[Byte]): CustomAvroClass = {

    val schema = CustomAvroClass.getClassSchema
    val reader = new SpecificDatumReader[CustomAvroClass](schema)
    val decoder = DecoderFactory.get.binaryDecoder(bytes, null)
    reader.read(null, decoder)
  }
```


#### Convert RDD to DataFrame

```
val jsonData: DStream[String] = records.map(avroRecord => avroRecord.toString)
jsonData.foreachRDD { stringRDD =>
      val sqlContext = SQLContext.getOrCreate(stringRDD.sparkContext)
      val recordsDF: DataFrame = sqlContext.read.json(stringRDD)  
    }
```
