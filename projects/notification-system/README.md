# 🔔 Project: Notification System

> **Learn by Building** - Multi-channel notification system using design patterns

## 📋 Requirements

- Support Email, SMS, and Push notifications
- User notification preferences
- Template-based messages
- Retry logic for failures
- Delivery tracking

## 🏗️ Design Patterns Used

- **Strategy Pattern**: Different notification channels
- **Observer Pattern**: Subscribe to events
- **Template Method**: Message formatting
- **Factory Pattern**: Create notifiers

## Key Implementation

```java
// Strategy Interface
public interface NotificationChannel {
    void send(Notification notification, User user);
    boolean isAvailable();
}

// Email Channel
public class EmailChannel implements NotificationChannel {
    private final EmailClient emailClient;
    
    @Override
    public void send(Notification notification, User user) {
        String html = formatAsHtml(notification);
        emailClient.send(user.getEmail(), notification.getSubject(), html);
    }
}

// SMS Channel
public class SmsChannel implements NotificationChannel {
    private final SmsGateway gateway;
    
    @Override
    public void send(Notification notification, User user) {
        String text = truncateMessage(notification.getContent(), 160);
        gateway.send(user.getPhoneNumber(), text);
    }
}

// Notification Service
public class NotificationService {
    private final Map<String, NotificationChannel> channels;
    private final UserPreferenceService preferences;
    private final RetryTemplate retryTemplate;
    
    public void notify(String userId, Notification notification) {
        Set<String> userChannels = preferences.getChannels(userId);
        User user = userService.getUser(userId);
        
        for (String channelType : userChannels) {
            NotificationChannel channel = channels.get(channelType);
            
            retryTemplate.execute(() -> {
                channel.send(notification, user);
                trackDelivery(notification, user, channelType, SUCCESS);
            }, exception -> {
                trackDelivery(notification, user, channelType, FAILED);
            });
        }
    }
}

// Event-based notifications (Observer)
public class OrderEventListener implements EventListener<OrderEvent> {
    private final NotificationService notificationService;
    
    @Override
    public void onEvent(OrderEvent event) {
        switch (event.getType()) {
            case ORDER_PLACED:
                sendOrderConfirmation(event.getOrder());
                break;
            case ORDER_SHIPPED:
                sendShippingNotification(event.getOrder());
                break;
        }
    }
}
```

## 🧪 Concepts Covered

- ✅ Strategy Pattern
- ✅ Observer Pattern
- ✅ Dependency Injection
- ✅ Retry Mechanism
- ✅ Event-Driven Design
