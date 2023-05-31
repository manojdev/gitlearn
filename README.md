import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.*;

import static org.mockito.Mockito.*;

class YourTestClass {
    @Mock
    private EventRepository eventRepository;

    @Mock
    private NotificationHttpCommunicator notificationHttpCommunicator;

    private Map<String, List<String>> topicCache;

    private YourClass yourClass;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        topicCache = new HashMap<>();
        yourClass = new YourClass(eventRepository, notificationHttpCommunicator, topicCache);
    }

    @Test
    void testProcessAliNotifications_withTopicInCache() {
        // Arrange
        List<Event> fetchedEvents = new ArrayList<>();
        Event eventObject = new Event("eventName", "domain", "environment");
        fetchedEvents.add(eventObject);
        List<String> topicIds = new ArrayList<>();
        topicIds.add("topicId");
        topicCache.put("eventName_domain-environment", topicIds);

        when(eventRepository.findAr10()).thenReturn(fetchedEvents);

        // Act
        yourClass.processAliNotifications();

        // Assert
        verify(notificationHttpCommunicator, never()).findTopic(any(Event.class));
        verify(notificationHttpCommunicator, times(1)).publishEvents("topicId", eventObject);
    }

    @Test
    void testProcessAliNotifications_withTopicNotInCache() {
        // Arrange
        List<Event> fetchedEvents = new ArrayList<>();
        Event eventObject = new Event("eventName", "domain", "environment");
        fetchedEvents.add(eventObject);
        List<ApiTopic> topicList = new ArrayList<>();
        ApiTopic topic = new ApiTopic("topicName", "domain", "environment");
        topicList.add(topic);
        List<String> topicIds = new ArrayList<>();
        topicIds.add("topicId");

        when(eventRepository.findAr10()).thenReturn(fetchedEvents);
        when(notificationHttpCommunicator.findTopic(eventObject)).thenReturn(topicList);
        when(notificationHttpCommunicator.publishEvents(anyString(), any(Event.class))).thenReturn(true);

        // Act
        yourClass.processAliNotifications();

        // Assert
        verify(notificationHttpCommunicator, times(1)).findTopic(eventObject);
        verify(notificationHttpCommunicator, times(1)).publishEvents("topicId", eventObject);
    }

    @Test
    void testProcessAliNotifications_withMaxCacheSize() {
        // Arrange
        List<Event> fetchedEvents = new ArrayList<>();
        Event eventObject = new Event("eventName", "domain", "environment");
        fetchedEvents.add(eventObject);
        List<ApiTopic> topicList = new ArrayList<>();
        ApiTopic topic = new ApiTopic("topicName", "domain", "environment");
        topicList.add(topic);
        List<String> topicIds = new ArrayList<>();
        topicIds.add("topicId");

        for (int i = 0; i < 1000; i++) {
            topicCache.put("key" + i, Collections.singletonList("value" + i));
        }

        when(eventRepository.findAr10()).thenReturn(fetchedEvents);
        when(notificationHttpCommunicator.findTopic(eventObject)).thenReturn(topicList);
        when(notificationHttpCommunicator.publishEvents(anyString(), any(Event.class))).thenReturn(true);

        // Act
        yourClass.processAliNotifications();

        // Assert
        verify(notificationHttpCommunicator, times(1)).findTopic(eventObject);
        verify(notificationHttpCommunicator, times(1)).publishEvents("topicId", eventObject);

        // Additional assertions for cache size
        int expectedCacheSize = 1000 - 200 + 1;
