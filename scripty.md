<script>
java
public void run() {
   StreamManager streamManager = StreamManager.create(controllerURI);

   final boolean scopeIsNew = streamManager.createScope(scope);
   StreamConfiguration streamConfig = StreamConfiguration.builder()
           .scalingPolicy(ScalingPolicy.fixed(1))
           .build();
   final boolean streamIsNew = streamManager.createStream(scope, streamName, streamConfig);

   final String readerGroup = UUID.randomUUID().toString().replace("-", "");
   final ReaderGroupConfig readerGroupConfig = ReaderGroupConfig.builder()
                                                                .stream(Stream.of(scope, streamName))
                                                                .build();
   try (ReaderGroupManager readerGroupManager = ReaderGroupManager.withScope(scope, controllerURI)) {
       readerGroupManager.createReaderGroup(readerGroup, readerGroupConfig);
   }

   try (ClientFactory clientFactory = ClientFactory.withScope(scope, controllerURI);
        EventStreamReader<String> reader = clientFactory.createReader("reader",
                                                                      readerGroup,
                                                     new JavaSerializer<String>(),
                                                  ReaderConfig.builder().build())) {
        System.out.format("Reading all the events from %s/%s%n", scope, streamName);
        EventRead<String> event = null;
        do {
           try {
               event = reader.readNextEvent(READER_TIMEOUT_MS);
               if (event.getEvent() != null) {
                   System.out.format("Read event '%s'%n", event.getEvent());
               }
           } catch (ReinitializationRequiredException e) {
               //There are certain circumstances where the reader needs to be reinitialized
               e.printStackTrace();
           }
       } while (event.getEvent() != null);
       System.out.format("No more events from %s/%s%n", scope, streamName);
   }
</script>

