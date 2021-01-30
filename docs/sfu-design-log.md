
# SFU design log

## 1-12-20 Allow subs to connect without ingress present

Few ways to do this:
- Don't do it, only allow subs to connect after pub: current mode
- Specify one node as 'root', only root allows subs without pub.
- root gives subs 3txs for video, but ONLY transmits idle-video on first of 3x
- when root ingress comes, if 3x simulcast, we fill the 3x tracks with video
- when root ingress comes. if just 1x video, we only send on first of 3x tracks
- this means no re-negotiation is necessary.
- This also means an SFU always creates 3x video tracks to a subscriber.
- It may not fill those tracks with media, depending upon what hits it's ingress,
but the tracks are ready if simulcast comes in, or if 3x non-simu comes in.


### how to do re-negotiation
- no apparent need to support this yet, but....
- use long polling
- use whip-like, offer from browser, answer from sfu



## 1 13 21 notes on tracks

Each subscriber has ZERO local tracks!
We stop using NewTrackLocalStaticRTP to create a track for each subscriber.
we use addtrack(audio), addtrack(video1), addtrack(video1),
and then replacetrack on keyframe before using WriteRTP to write the RTP to a particulae track

## 1 14 21 notes on track switching

Subscribers do NOT get their own local tracks.
Subscribers do have an RtpSender for each track towards the sub.
Browser's just get a single track towards them.
We switch media towards the browser by checking each RX packet on each RX track.
When it is a true keyframe, AND there are sub's requesting this RX track, then we use 
 pc.ReplaceTrack() before we write the media to the local track.
This should ensure that the sub will start receiving media from the localtrack it is waiting
to join.

## 1 14 21 notes on number of channels for subscribers to sfu

When browsers subscribe to sfu, they offer 1 m=video recvonly section
When sfus subscribe to another sfu, they offer 3 m=video recvonly section
The upstream sfu, uses the number of offered receivers 

## 1 14 21 notes on why the 'issfu' param is not necessary

`&sfu=1` was a query param from sub to upstream sfu. it indicated to 
the upstream how many tracks it should push to the downstream.
it turns out the number of m=video sections is a cleaner way to do this.

## 1 29 21 Why we need a splicer in front of each Pion video track

If we only ever had a single RTP source for each RTP destination, 
RTP splicers would not be necessary
This would look like: 
- Ingest could only connect a single time for RTP continuity
- Restart would be necessary for new ingress connection
- (Or all subscribers would be required to re-connect for new ingress)
- No Idle video loop would be possible, as this requires switching between ingress and idle loop RTP stream.
- Also, without RTP splicing, A/B/C simulcast switching is basically impossible.

Thus, to support these three features (ingress reconnect, idle-loop, simulcast switchin)


