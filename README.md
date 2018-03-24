# Semantic Image Segmentation Web Service

This service is built atop [DeepLab-ResNet-TensorFlow](https://github.com/DrSleep/tensorflow-deeplab-resnet/tree/crf) implementation, branch **crf**. For detailed information, please refer to the README of this repo.

## Dependencies

* install [TensorFlow](https://www.tensorflow.org/install/install_linux) (as of 3/17/2018 production, TensorFlow 1.5.0 was used)
* install requirements:

```
pip install -r requirements.txt
```

* download pre-trained weights from [here](https://drive.google.com/drive/folders/0B_rootXHuswsZ0E4Mjh1ZU5xZVU) (author's link) or ask *peter [at] remap [dot] ucla [dot] edu* if original link doesn't work anymore and **save it to the repo directory**;

> **NOTE** `deeplab_resnet.ckpt` should be used.

* in repo directory, create folders:

```
mkdir upload results
```

* install tornado:

```
pip install tornado
```

## Configuration

Since I didn't have time to figure out how to get IP address from within the running tornado service, modify [tornado/run.py](tonrado/run.py#L34) to your current public IP of the machine where this service will run.

## Run

Start service:

```
python tornado/run.py
```

## Check

Go to `http://<ip-address>:8888` - you should be able to see upload page where you can upload an image for segmentation. Once uploaded, service will return URL which you'll need to poll for the result (or it will return **code 423** if service is busy processing previous upload).

## API

* `/result/<FILE-ID>` - returns 404 if result was not processed yet or not found or returns segmented image (PNG);
* `/status` - returns **ok** if service runs normally.

## Handy 

* `curl` command for uploading images:

```
curl -F "file=@<path to file>;" http://<ip-address>:8888/upload
```

* python code for uploading images, checking the result and saving it into a file:

```
import requests
from time import sleep

port = 8888
ipaddress = <ip-address>
hostUrl = "http://"+ipaddress+":"+str(port)

uploadUrl = hostUrl+"/upload"

def upload(fname):
	imageFile = {'file': open(fname, 'rb')}
	response = requests.post(uploadUrl, files=imageFile)
	if response.status_code == 200:
		print("Upload successful.")
		statusCode = 0
		it = 0
		maxIter = 10
		maxWait = 2000
		while statusCode != 200 and it < maxIter:
			print("Fetching result "+str(it+1)+"/"+str(maxIter)+"...")
			r = requests.get("http://127.0.0.1:8888/result/fabd1020-a705-463d-b386-c60048d20c1a", stream=True)
			statusCode = r.status_code
			it += 1
			if statusCode != 200:
				sleep(float(maxWait)/float(maxIter))

		if statusCode == 200:
			fname = "./result.png"
			with open(fname, 'wb') as f:
				for chunk in r:
					f.write(chunk)
			print("saved result at "+fname)
		else:
			print("time out receiving result from the server")

def main():
	upload("/Users/peetonn/Downloads/img12.png")

if __name__ == "__main__":
	main()
```
