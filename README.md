# vod-upload-sdk-demo

## 1) Include JS, https://libs.lexo.video/js/put-video-sdk.min.js
```html
<script src="https://libs.lexo.video/js/put-video-sdk.min.js"></script>
```

## 2) Initialize PV
```js
const pv = new PV({
    getAuthorization: function (options, callback) {
        const url = '/get_sts?key={key}'; 
//Replace the given URL with your backend service's URL and obtain a temporary key for the callback. Alternatively, you can also utilize the whitelist feature. IPs in this whitelist can successfully make a call even without a key.
        const xhr = new XMLHttpRequest();
        let data = null;
        let credentials = null;
        xhr.open('GET', url, true);
        xhr.onload = function (e) {
            try {
                data = JSON.parse(e.target.responseText);
                credentials = data.credentials;
            } catch (e) {
            }
            if (!data || !credentials) {
                return console.error('Invalid credentials:\n' + JSON.stringify(data, null, 2))
            };
            console.log('Uploading', JSON.stringify(options));
            console.log(credentials); // Check the format of credentials
            callback({
                TmpSecretId: credentials.tmpSecretId,
                TmpSecretKey: credentials.tmpSecretKey,
                SecurityToken: credentials.sessionToken,
                // It is suggested to return server time as the signature start time to avoid signature errors due to significant local time deviations on user's browser
                StartTime: data.startTime, // Timestamp in seconds, e.g.: 1580000000
                ExpiredTime: data.expiredTime, // Timestamp in seconds, e.g.: 1580000000
            });
        };
        xhr.send();
    }
});
```

## 3) Call the pv's uploadFile method
```js
// Listen for the selected file
document.getElementById('file-selector').onchange = function () {
    var body = this.files[0];
    if (!body) return;
    pv.uploadFile({
        file_id: "ddd-f3ew2fwe-fewqcew-cewcwe",
        os_id: "os_id", // Upload ID tag, assigned by Lexo
        // task_type:
        // A: MP4 720 -> HLS 480 720 1080; (transcoding + upscaling + compression)
        // C: MP4 -> MP4 (compression)
        task_type: "A",
        Body: body,
        SliceSize: 1024 * 1024 * 4, // If the file is larger than 4MB, upload in chunks
        onTaskReady: function (tid) {
            task_id = tid;
        },
        onProgress: function (progressData) {
            console.log('Uploading', JSON.stringify(progressData));
        },
        onFileFinish: function(progressData) {
            console.log('Succeeded', JSON.stringify(progressData));
        }
    }, function (err, data) {
        console.log(err, data);
    });
    // Tasks can be paused and restarted using a queue
    // cos.pauseTask(taskId);
}
```
