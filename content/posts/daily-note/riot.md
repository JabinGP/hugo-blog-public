(engine.go-Index) -> (engine.go-internalIndexDoc) -> 发送segmenterReq到engine.segmenterChan

(segment.go-segmenterWorker) watching engine.segmenterChan, when he get request from chan, he sent it to (engine.rankerAddDocChans[shard]), (shard) is from (engine.getShard(request.hash)) which used to do a hash.

(ranker_worker.go - rankerAddDoc) watching engine.rankerAddDocChans[shard], when it get request, call (engine.rankers[shard].AddDoc)

(ranker.go - AddDoc) add doc into (ranker.go - ranker.lock.content[docId])


back to (engine.go - Index), send (storeIndexDocReq) to (engine.storeIndexDocChans[hash])

(store_worker.go - storeIndexDoc), watching engine.storeIndexDocChans[shard], when it get request, call (engine.dbs[shard].Set(b, buf.Bytes()))

