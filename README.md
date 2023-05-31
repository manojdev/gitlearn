import okhttp3.mockwebserver.MockResponse;
import okhttp3.mockwebserver.MockWebServer;
import okhttp3.mockwebserver.RecordedRequest;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.web.reactive.function.client.WebClient;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class NotificationHttpCommunicatorTest {
    private MockWebServer server;
    private NotificationHttpCommunicator communicator;

    @BeforeEach
    public void setup() throws IOException {
        server = new MockWebServer();
        server.start();

        WebClient.Builder webClientBuilder = WebClient.builder()
                .baseUrl(server.url("/").toString());

        WebClient client = webClientBuilder.build();
        URIBuilder uriBuilder = new URIBuilder();

        communicator = new NotificationHttpCommunicator(client, uriBuilder);
    }

    @AfterEach
    public void tearDown() throws IOException {
        server.shutdown();
    }

    @Test
    public void testGetEvents() throws InterruptedException {
        // Enqueue a mock response from the server
        List<EventDTO> eventDtoList = Arrays.asList(new EventDTO(), new EventDTO());
        server.enqueue(new MockResponse()
                .setBody(JsonUtils.toJson(eventDtoList)));

        // Call the method under test
        List<EventDTO> result = communicator.getEvents("start", "end");

        // Verify the request sent to the server
        RecordedRequest recordedRequest = server.takeRequest();
        assertEquals("/events?start=start&end=end", recordedRequest.getPath());

        // Verify the result
        assertEquals(eventDtoList, result);
    }
}
