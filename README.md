import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.messaging.simp.SimpMessagingTemplate;

class WebSocketServiceTest {

    private WebSocketService webSocketService;
    private SimpMessagingTemplate messagingTemplate;

    @BeforeEach
    void setUp() {
        messagingTemplate = Mockito.mock(SimpMessagingTemplate.class);
        webSocketService = new WebSocketService(messagingTemplate);
    }

    @Test
    void pushNotification_UserNotification_Success() {
        Long userId = 123L;
        UserNotification userNotification = new UserNotification();
        // Set up any required data in userNotification

        webSocketService.pushNotification(userId, userNotification);

        // Verify that messagingTemplate.convertAndSendToUser was called with the expected arguments
        Mockito.verify(messagingTemplate).convertAndSendToUser(
            Mockito.eq(userId.toString()),
            Mockito.eq("/Live/notifications"),
            Mockito.eq(userNotification)
        );
    }

    @Test
    void pushNotification_UserNotificationEntity_Success() {
        Long userId = 123L;
        UserNotificationEntity userNotificationEntity = new UserNotificationEntity();
        // Set up any required data in userNotificationEntity

        webSocketService.pushNotification(userId, userNotificationEntity);

        // Verify that messagingTemplate.convertAndSendToUser was called with the expected arguments
        Mockito.verify(messagingTemplate).convertAndSendToUser(
            Mockito.eq(userId.toString()),
            Mockito.eq("/Live/notifications"),
            Mockito.any(UserNotification.class) // You can also verify that the correct UserNotification is sent
        );
    }
}
