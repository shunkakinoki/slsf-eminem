# PyTorch Serverless

[FastAI](http://www.fast.ai) PyTorch Serverless API (w/ AWS Lambda)

## Setup

- Install [Serverless Framework](https://serverless.com/) via npm

```bash
npm i -g serverless@v1.27.3
```

- Install python requirements plugin

```bash
sls plugin install -n serverless-python-requirements
```

## Configuration

- Setup your model in `lib/models.py` so that it can be imported by the handler in `api/predict.py` as a method

- Define your class labels in `lib/labels.txt` with one label per line, for example:

```txt
cat
dog
```

- Setup an [AWS CLI profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html) if you
don't have one already

- Create an [S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html#create-bucket-intro) that your
profile can access and upload your state dictionary

- Configure the `serverless.yml`

```bash
### Change service name to whatever you please
service: pytorch-serverless

provider:
    ...
    ### set this to your deployment stage
    stage: dev

    ### set this to your aws region
    region: us-west-2

    ### set this to your aws profile
    profile: slsadmin

    ### set this as needed between 128 - 3008, in 64mb intervals
    memorySize: 2048

    ### set this as needed (max 300)
    timeout: 120
    ...

    environment:
        ### set this to your S3 bucket name
        BUCKET_NAME: pytorch-serverless

        ### set this to your state dict filename
        STATE_DICT_NAME: dogscats-resnext50.h5

        ### set this to your input image size
        IMAGE_SIZE: 224

        ### set this to your image normalization stats
        IMAGE_STATS: ([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])
        ...

    variables:
        ### set this to your api version
        api_version: v0.0.1
```

## Invoke Local

Run function locally with params defined in `tests/predict_event.json`

```bash
AWS_PROFILE=yourProfile sls invoke local -f predict -p tests/predict_event.json
```

## Deployment

**Make sure [Docker](https://docs.docker.com/install/) is running**

Deploy to AWS Lambda

```bash
sls deploy -v
```

## Endpoints

### **GET** `/predict`

Return prediction for a single image.

- **Headers**

```json
(required)
X-API-KEY=[string]   ### Your generated API Key
```

- **URL Parameters**

```json
(required)
image_url=[url]   ### URL of image to classify

(optional)
top_k=[integer]   ### Number of top results (default: 3)
```

- **Success Response (200)**

```json
{
    "predictions": [
        {

            "label": "dog",
            "log": -0.00004426980376592837,
            "prob": 0.9999557137489319
        },
        {

            "label": "cat",
            "log": -10.025229454040527,
            "prob": 0.0000442688433395233
        }
    ]
}
```

- **Error Response (500)**

```json
{
    "error": "Something went wrong...",
    "traceback": "..."
}
```

## Logs

Tail logs to console

```bash
sls logs -f predict -t
```
