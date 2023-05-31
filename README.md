import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.web.reactive.function.client.WebClient;

import java.util.Arrays;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

public class NotificationHttpCommunicatorTest {
    @Mock
    private WebClient client;

    @Mock
    private URIBuilder uriBuilder;

    private NotificationHttpCommunicator communicator;

    @BeforeEach
    public void setup() {
        MockitoAnnotations.openMocks(this);
        communicator = new NotificationHttpCommunicator(client, uriBuilder);
    }

    @Test
    public void testGetEvents() {
        // Mock the WebClient's response
        List<EventDTO> eventDtoList = Arrays.asList(new EventDTO(), new EventDTO());
        WebClient.ResponseSpec responseSpec = Mockito.mock(WebClient.ResponseSpec.class);
        when(responseSpec.bodyToMono(any(ParameterizedTypeReference.class)))
                .thenReturn(Mono.just(eventDtoList));
        when(client.get()).thenReturn(responseSpec);

        // Mock the URIBuilder
        when(uriBuilder.buildGetAllEvents("start", "end")).thenReturn("http://example.com/events");

        // Call the method under test
        List<EventDTO> result = communicator.getEvents("start", "end");

        // Verify the result
        assertEquals(eventDtoList, result);
    }
}
