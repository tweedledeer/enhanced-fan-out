1. Register consumer for K-Stream for enhanced fan out:
   aws kinesis register-stream-consumer --consumer-name efo_consumer --stream-arn arn:aws:kinesis:eu-central-1:<stg_account_id>:stream/<stream_name>
   
Response: {
   "Consumer": {
   "ConsumerName": "efo_consumer",
   "ConsumerARN": "arn:aws:kinesis:eu-central-1:<stg_account_id>:stream/<stream_name>/consumer/<consumer_name>:<consumer_id>",
   "ConsumerStatus": "CREATING",
   "ConsumerCreationTimestamp": "2022-12-27T19:06:26+02:00"
   }
}

2. Let's populate records from bucket to Kinesis via python code (console might be used):

import boto3

region = 'eu-central-1'

stream_name = 'stream-tst'

kinesis = boto3.client("kinesis", region_name=region)

s3 = boto3.client('s3')

bucket = '<some_bucket>'

result = s3.list_objects(Bucket=bucket, Prefix='gk_test/input')

data = s3.get_object(Bucket=bucket, Key=result.get('Contents')[1].get('Key'))

contents = data['Body'].read()

#Put record to stream

kinesis.put_record(StreamName=stream_name, Data=contents, PartitionKey="A")

3. Run src/main/java/org/example/SubscribeToShardSimpleImpl.java via IDE or using jar file (
   java -jar target/EFO_mvn-1.0-SNAPSHOT.jar ) and observe
that on putting records to K-stream's consumer that've registered for enhanced fan out
these records are getting read.
