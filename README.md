# ChimeDig
This Repository contains the Code debug points.Learned to Create a Custom Video Calling App Using Amazon Chime SDK.

### Code Debug

##### 1.Show Video Thumbnail upon user video turn on and Off of the video (Without updating the tile)
- onVideoTile Added , add the isVideoOnParam in the VideoCollectionTile data class.
``



        var isFirstTime:Boolean = false
// LOGIC STARTS HERE
        if(!meetingModel.isCameraOn &&tileState.isLocalTile && !isFirstTime){
        // ONLY ADD THE LOCAL TILE ONCE.
            videoList.add(createVideoCollectionTile(tileState))
            isFirstTime=true
        }else{
            for (index in videoList.indices){
                if(videoList.get(index).videoTileState.attendeeId!=tileState.attendeeId){
                    //NEW TILE ADDED
                    videoList.add(createVideoCollectionTile(tileState))
                }else{
                    // SAME TILE WE NEED TO UPDATE
                    // do not add anything.
                    var videoCollectionTileTemp = createVideoCollectionTile(tileState) //CURRENT TILE FROM SDK
                    var videoCollectionNeededUpdate = videoList.get(index) // TILE WE NEED TO UPDATE.
                    //Updating the tile
                    meetingModel.updateVideoStatesInCurrentPage()
                    videoCollectionNeededUpdate.videoRenderView = videoCollectionTileTemp.videoRenderView
                    videoCollectionNeededUpdate.isVideoOn = true
                    videoCollectionNeededUpdate.videoTileState = videoCollectionTileTemp.videoTileState
                    videoCollectionNeededUpdate.attendeeName = videoCollectionTileTemp.attendeeName
                    videoCollectionNeededUpdate.pauseMessageView = videoCollectionTileTemp.pauseMessageView

                    // Updating the list
                    videoList.set(index,videoCollectionNeededUpdate)
                    videoTileAdapter.notifyItemRemoved(index)
                    videoTileAdapter.notifyItemInserted(index)
                    videoTileAdapter.notifyItemChanged(index,videoCollectionNeededUpdate)
                    break

                }
            }

        }
``
- Change in the onVideoTileRemoved observer ::
```
 //NEW CODE
        val tileId: Int = tileState.tileId
        var videoCollectionTile : VideoCollectionTile
//        audioVideo.unbindVideoView(tileState.tileId)

        var videoCollectionTileTemp = createVideoCollectionTile(tileState)
        videoCollectionTileTemp.isVideoOn=false


//LOGIC STARTS HERE
        for (index in videoList.indices){
            if(videoList.get(index).videoTileState.tileId==tileState.tileId){
                // ALREADY ADDED TILE JUST PAUSE THE VIDEO
                videoList.get(index).isVideoOn = false
//                audioVideo.pauseRemoteVideoTile(tileState.tileId)
                videoTileAdapter.notifyDataSetChanged()
            }
        }

```


- Change in the adapter class. Add this check where your are binding the video view
```
if(videoCollectionTile.isVideoOn){
                    binding.videoRenderView.visibility = View.VISIBLE
                    audioVideo.bindVideoView(binding.videoRenderView, videoCollectionTile.videoTileState.tileId)
                    binding.ivVideoOff.visibility = View.GONE // CUSTOM THUMBNAIL IMAGE.
                }else{
                    audioVideo.unbindVideoView(videoCollectionTile.videoTileState.tileId)
                    binding.videoRenderView.visibility = View.GONE
                    binding.ivVideoOff.visibility = View.VISIBLE
                }
```
