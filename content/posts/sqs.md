+++
author = "febin.stephen"
title = "An async SQS consumer in Python"
date = "2023-06-10"
description = "My coding lessons."
tags = [
    "aws",
    "sqs",
    "python"
]
weight = 10
+++

Simple Queue Service (SQS) is a message queue service provided by AWS.<!--more--> SQS works by the push and poll model. It helps applications implement decoupling and is highly scalable. We can choose between a standard queue or FIFO queue based on your use-cases. Both of them comes with unique features. AWS allows unlimited queues and messages in any region. Each message payload can be a maximum size of 256 KB of text in any format and each 64 KB is billed as 1 request. We can also sent, recieve and delete messages in batches of upto 10 messages or 256 KB. With long polling enabled the queue waits for a given time when it is empty before sending new messages, this can minimize cost while recieving new messages as quickly as possible. The maximum reention period we can set for a message is 14 days. With message locking functionality SQS ensures that each message is consumed only once. The messages are encrypted before storing into the queue. Unprocessed messages can be moved to a dead letter queue, developers can inspect DLQ to understand why they are not processed and once remediated they can move them back to original queue.
 

| Standard  | FIFO |
| ----- | --- |
| Unlimited throughput: supports unlimited transactions per second per API Action   | By default FIFO supports 3000 messages per second with batching and 300 messages per second without batching   |
| At-least-once-delivery | Exactly-once processing  |
| Best-Effort Ordering | First-in-First-out delivery  |
| At-least-once-delivery | Exactly-once processing  |
| Most suited for applications that can handle unordered and multiple message delivery | Most suited for applications where order of messages are critical and the can't tolerate the duplicates  |

SQS can be used with many other AWS services to make your distributed applications more scalable and reliable. There are many design patterns to leverage SQS for scalability and reliability in your application.

In this example I used **localstack** to setup an aws environment in local machine and created an sqs queue. You can check the following link to learn how to install localstack and provision aws resources locally [Getting started](https://docs.localstack.cloud/getting-started/). With localstack you do not need to create an aws account nor provision any resources and worry about the billitng. It reduces aws spend and remove the complexity of maintaining multiple aws dev accounts.

The following script will start running two consume_messages tasks concurrently receiving and processing messages from the SQS queue. **aiobotocore** is an async client for amazon services using botocore and aiohttp/asyncio which is used here as the context manager for accessing and processing the queue. The secret and access keys values passed in the create_client function does not need to be your original keys, it can be any dummy values.

```

import asyncio
import json
import sys
import logging

import botocore.exceptions
import pydantic
from aiobotocore.session import get_session

from sqs_consumer_project.setup import setup_logging


async def consume_messages(queue_name: str, shutdown_signal: asyncio.Event):
    async with get_session().create_client(
        "sqs",
        region_name="us-east-1",
        endpoint_url="http://localhost:4566",
        aws_access_key_id="test",
        aws_secret_access_key="test",
    ) as client:
        try:
            get_url_response = await client.get_queue_url(QueueName=queue_name)
        except botocore.exceptions.ClientError as err:
            if err.response["Error"]["Code"] == "AWS.SimpleQueueService.NonExistentQueue":
                logging.error(f"Queue {queue_name} does not exist")
                sys.exit(1)
            else:
                raise

        queue_url = get_url_response["QueueUrl"]

        while not shutdown_signal.is_set():
            logging.info("Polling for messages")
            try:
                receive_message_response = await client.receive_message(
                    QueueUrl=queue_url,
                    MaxNumberOfMessages=1,
                    WaitTimeSeconds=2,
                )

                if "Messages" in receive_message_response:
                    logging.info(
                        "receive_messages got messages",
                        extra={"message_count": len(receive_message_response["Messages"])},
                    )
                    for msg in receive_message_response["Messages"]:
                        message_id = msg["MessageId"]
                        message_body = msg["Body"]
                        successfully_processed = await message_processor(message_id, message_body)

                        if successfully_processed:
                            # Need to remove msg from queue or else it'll reappear, you could see this by
                            # checking ApproximateNumberOfMessages and ApproximateNumberOfMessagesNotVisible
                            # in the queue.
                            await client.delete_message(
                                QueueUrl=queue_url,
                                ReceiptHandle=msg["ReceiptHandle"],
                            )
                        else:
                            logging.error("Failed to process message", extra={"message_id": message_id})
                else:
                    logging.debug("No messages in queue")
            except asyncio.CancelledError:
                logging.error("Cancel Error")
                break

        logging.info("Finished")

async def main():
    setup_logging()

    queue_name = "my-queue2"
    consumer_count = 2
    shutdown_signal = asyncio.Event()
    consumers = [consume_messages(queue_name, shutdown_signal) for _ in range(consumer_count)]
    await asyncio.gather(*consumers)


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        logging.info("Script interrupted by user")

```

After receiving each message you can use it to perform your specific actions like sending user notifications or update the database. The `message_processor` and `do_work` functions does not do anything complex logic here except validating the message format and log the details, but you should modify them based on your use-case.

```

async def message_processor(message_id: str, message_body: str) -> bool:
    logging.info("Starting MessageId processing", extra={"message_id": message_id})
    try:
        message_dict = json.loads(message_body)
        logging.info("pydantic model", extra={"sqs_message": message_dict})
    except pydantic.ValidationError as e:
        logging.error("Invalid message format", extra={"error": e})
        return False

    return await do_work(message_dict)


async def do_work(message) -> bool:
    logging.info("Started work", extra={"body": message})

    logging.info(
        "Completed work",
        extra={"body": message}
    )
    return True

```



{{< css.inline >}}

<style>
.canon { background: white; width: 100%; height: auto; }
</style>

{{< /css.inline >}}
