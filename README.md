import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

public class NotificationHttpCommunicatorTest {
    @Mock
    private NotificationHttpCommunicator communicator;

    @Mock
    private EventRepository eventRepository;

    @Mock
    private JobRepository jobRepository;

    @BeforeEach
    public void setup() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testPostEvents_Success() {
        // Create test data
        String topic = "topic";
        Event eventObject = new Event();
        EventRequestDTO eventRequest = new EventRequestDTO();
        eventRequest.setData(eventObject.getData());

        // Mock the response
        ResponseEntity<ApiEvent> response = new ResponseEntity<>(HttpStatus.OK);
        when(communicator.postEvents(eventRequest, topic)).thenReturn(response);

        // Call the method under test
        communicator.postevents(topic, eventObject);

        // Verify the interactions
        verify(eventRepository, times(1)).delete(eventObject);
    }

    @Test
    public void testPostEvents_TopicNotFound() {
        // Create test data
        String topic = "topic";
        Event eventObject = new Event();
        EventRequestDTO eventRequest = new EventRequestDTO();
        eventRequest.setData(eventObject.getData());

        // Mock the response
        ResponseEntity<ApiEvent> response = new ResponseEntity<>(HttpStatus.NOT_FOUND);
        when(communicator.postEvents(eventRequest, topic)).thenReturn(response);

        // Call the method under test
        communicator.postevents(topic, eventObject);

        // Verify the interactions
        verifyNoInteractions(eventRepository);
    }

    @Test
    public void testUpdateLastRun_JobListNotEmpty() {
        // Create test data
        LocalDateTime start = LocalDateTime.now();
        LocalDateTime end = LocalDateTime.now();
        List<Job> jobList = new ArrayList<>();
        jobList.add(new Job());

        // Mock the job repository
        when(jobRepository.findAll()).thenReturn(jobList);
        doNothing().when(jobRepository).save(any(Job.class));

        // Call the method under test
        communicator.updateLastRun(start, end);

        // Verify the interactions
        verifyNoMoreInteractions(jobRepository);
    }

    @Test
    public void testUpdateLastRun_JobListEmpty() {
        // Create test data
        LocalDateTime start = LocalDateTime.now();
        LocalDateTime end = LocalDateTime.now();
        List<Job> jobList = new ArrayList<>();

        // Mock the job repository
        when(jobRepository.findAll()).thenReturn(jobList);
        doNothing().when(jobRepository).save(any(Job.class));

        // Call the method under test
        communicator.updateLastRun(start, end);

        // Verify the interactions
        verify(jobRepository, times(1)).save(any(Job.class));
    }
}
