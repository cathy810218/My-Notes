# Merge Video and Audio (iOS9 Swift)

Import libraries
````Swift
import AVFoundation
import AVKit
````

## AVMutableComposition

Initialize AVMutableComposition for adding video and audio tracks

`let mixComposition = AVMutableComposition()`


## AVURLAsset

Create audio and video assets 

````Swift
audioAsset = AVURLAsset(URL: audioURL)
videoAsset = AVURLAsset(URL: videoURL)
````


## AVMutableCompositionTrack

Create time range. In this case, `audioTimeRange` is set to be the length of video,
so if the length of the video is shorter, then it will cut off the audio once the video ends.

````Swift
let audioTimeRange = CMTimeRangeMake(kCMTimeZero, videoAsset.duration)
let videoTimeRange = CMTimeRangeMake(kCMTimeZero, videoAsset.duration)
````

Create two tracks: VideoTrack and AudioTrack and add both tracks to the mixComposition

````Swift
// audioTrack
let audioTrack: AVMutableCompositionTrack = mixComposition.addMutableTrackWithMediaType
(AVMediaTypeAudio, preferredTrackID: kCMPersistentTrackID_Invalid)

try! audioTrack.insertTimeRange(audioTimeRange, ofTrack: audioAsset.tracksWithMediaType
(AVMediaTypeAudio)[0], atTime: kCMTimeZero)
        
// videoTrack
let videoTrack: AVMutableCompositionTrack = mixComposition.addMutableTrackWithMediaType
(AVMediaTypeVideo, preferredTrackID: kCMPersistentTrackID_Invalid)

try! videoTrack.insertTimeRange(videoTimeRange, ofTrack: videoAsset.tracksWithMediaType
(AVMediaTypeVideo)[0], atTime: kCMTimeZero)
````


## Create output path

````swift
let dirPaths = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true)
let docsDir = dirPaths[0]
let outputFilePath = docsDir.stringByAppendingString("/FinalVideo.mov")
let outputFileURL = NSURL.fileURLWithPath(outputFilePath)
````

Remove the old video if there's one
````swift
if (NSFileManager.defaultManager().fileExistsAtPath(outputFilePath)) {
    try! NSFileManager.defaultManager().removeItemAtPath(outputFilePath)
}
````


## AVAssetExportSession
Export the video with output file mov.

````Swift
let export =  AVAssetExportSession(asset: mixComposition, presetName: AVAssetExportPresetHighestQuality)
export?.outputFileType = AVFileTypeQuickTimeMovie
export?.outputURL = outputFileURL
        
export?.exportAsynchronouslyWithCompletionHandler({ 
    dispatch_async(dispatch_get_main_queue(), { 
        self.exportDidFinish(export!)
        completionBlock(outputFileURL)
    })
})
````


## AVAssetExportSession + PHPhotoLibrary
Export the final video to the photo album

````swift
    func exportDidFinish(session: AVAssetExportSession) {
        if (session.status == AVAssetExportSessionStatus.Completed) {
            PHPhotoLibrary.sharedPhotoLibrary().performChanges({ 
                let createAssetRequest = PHAssetChangeRequest.creationRequestForAssetFromVideoAtFileURL(session.outputURL!)
                self.ph = (createAssetRequest?.placeholderForCreatedAsset)!
                
                }, completionHandler: { (assetURL, error) in
                    
                   dispatch_async(dispatch_get_main_queue(), { 
                        if (error != nil) {
                            print("ERROR")
                        } else {
                            print("SUCCESS!")
                        }
                   })
            })
        }
        audioAsset = nil
        videoAsset = nil
    }
````

