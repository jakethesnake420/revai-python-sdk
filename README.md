# Rev.ai Python SDK

[![Build Status](https://travis-ci.org/revdotcom/revai-python-sdk.svg?branch=develop)](https://travis-ci.org/revdotcom/revai-python-sdk)

## Documentation

See the [API docs](https://www.rev.ai/docs) for more information about the API and
more python examples.

## Installation

You don't need this source code unless you want to modify the package. If you just
want to use the package, just run:

    pip install --upgrade rev_ai

Install from source with:

    python setup.py install

### Requirements

- Python 2.7+ or Python 3.4+

## Usage

All you need to get started is your Access Token, which can be generated on
your [Settings Page](https://www.rev.ai/settings). Create a client with the
given Access Token:

```python
from rev_ai import apiclient

# create your client
client = apiclient.RevAiAPIClient("ACCESS TOKEN")
```

### Sending a file

Once you've set up your client with your Access Token sending a file is easy!

```python
# you can send a local file
job = client.submit_job_local_file("FILE PATH")

# or send a link to the file you want transcribed
job = client.submit_job_url("https://example.com/file-to-transcribe.mp3")
```

`job` will contain all the information normally found in a successful response from our
[Submit Job](https://www.rev.ai/docs#operation/SubmitTranscriptionJob) endpoint.

If you want to get fancy, both send job methods take `metadata`, `callback_url`,
`skip_diarization`, `skip_punctuation`, `speaker_channels_count` and `custom_vocabularies` as optional parameters, these are described in the request body of
the [Submit Job](https://www.rev.ai/docs#operation/SubmitTranscriptionJob) endpoint.

### Checking your file's status

You can check the status of your transcription job using its `id`

```python
job_details = client.get_job_details(job.id)
```

`job_details` will contain all information normally found in a successful response from
our [Get Job](https://www.rev.ai/docs#operation/GetJobById) endpoint

### Checking multiple files

You can retrieve a list of transcription jobs with optional parameters

```python
jobs = client.get_list_of_jobs()

# limit amount of retrieved jobs
jobs = client.get_list_of_jobs(limits=3)

# get jobs starting after a certain job id
jobs = client.get_list_of_jobs(starting_after='Umx5c6F7pH7r')
```

`jobs` will contain a list of job details having all information normally found in a successful response
from our [Get List of Jobs](https://www.rev.ai/docs#operation/GetListOfJobs) endpoint

### Deleting a job

You can delete a transcription job using its `id`

```python
client.delete_job(job.id)
```

 All data related to the job, such as input media and transcript, will be permanently deleted.
 A job can only by deleted once it's completed (either with success or failure).

### Getting your transcript

Once your file is transcribed, you can get your transcript in a few different forms:

```python
# as text
transcript_text = client.get_transcript_text(job.id)

# as json
transcript_json = client.get_transcript_json(job.id)

# or as a python object
transcript_object = client.get_transcript_object(job.id)
```

Both the json and object forms contain all the formation outlined in the response
of the [Get Transcript](https://www.rev.ai/docs#operation/GetTranscriptById) endpoint
when using the json response schema. While the text output is a string containing
just the text of your transcript

### Getting captions output

You can also get captions output from the SDK. We offer both SRT and VTT caption formats.
If you submitted your job as speaker channel audio then you must also provide a `channel_id` to be captioned:

```python
captions = client.get_captions(job.id, content_type=CaptionType.SRT, channel_id=None)
```

### Streamed outputs

Any output format can be retrieved as a stream. In these cases we return the raw http response to you. The output can be retrieved via `response.content`, `response.iter_lines()` or `response.iter_content()`.

```python
text_stream = client.get_transcript_text_as_stream(job.id)

json_stream = client.get_transcript_json_as_stream(job.id)

captions_stream = client.get_captions_as_stream(job.id)
```

## Streaming audio

In order to stream audio, you will need to setup a streaming client and a media configuration for the audio you will be sending.

```python
from rev_ai.streamingclient import RevAiStreamingClient
from rev_ai.models import MediaConfig

#on_error(error)
#on_close(code, reason)
#on_connected(id)

config = MediaConfig()
streaming_client = RevAiStreamingClient("ACCESS TOKEN",
                                        config,
                                        on_error=ERRORFUNC,
                                        on_close=CLOSEFUNC,
                                        on_connected=CONNECTEDFUNC)
```

`on_error`, `on_close`, and `on_connected` are optional parameters that are functions to be called when the websocket errors, closes, and connects respectively. The default `on_error` raises the error, `on_close` prints out the code and reason for closing, and `on_connected` prints out the job ID.
If passing in custom functions, make sure you provide the right parameters. See the sample code for the parameters.

Once you have a streaming client setup with a `MediaConfig` and access token, you can obtain a transcription generator of your audio.

```python
response_generator = streaming_client.start(AUDIO_GENERATOR)
```

`response_generator` is a generator object that yields the transcription results of the audio including partial and final transcriptions. The `start` method creates a thread sending audio pieces from the `AUDIO_GENERATOR` to our 
[streaming] endpoint.

If you want to end the connection early, you can!

```python
streaming_client.end()
```

Otherwise, the connection will end when the server obtains an "EOS" message.