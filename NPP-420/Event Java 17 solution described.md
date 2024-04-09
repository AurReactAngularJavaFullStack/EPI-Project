D'accord, il semble que vous cherchiez à créer un service en Java qui valide les données entrantes et les insère dans une base de données. Je vais esquisser une solution basée sur les images que vous avez partagées.

Vous aurez besoin de plusieurs composants pour mettre en place cette solution :

1. Une entité JPA pour représenter les données de l'événement.
2. Un repository JPA pour interagir avec la base de données.
3. Un service pour gérer la logique métier.
4. Un gestionnaire d'exceptions pour gérer les erreurs de validation des données.

Voici un exemple de code Java 17 qui intègre tous ces éléments :

Ce code est une ébauche générale basée sur les images fournies. La fonction `validateEventDTO` doit être implémentée avec la logique de validation spécifique illustrée dans vos images. La classe `CustomException` peut être étendue pour gérer différentes exceptions personnalisées.

Le code ci-dessus n'est pas complet et nécessite une implémentation détaillée pour chaque méthode et gestion d'erreur basée sur les règles de gestion fournies dans vos images. Vous devez remplir les détails pour la méthode de validation, la conversion DTO et le mapping, ainsi que la gestion des exceptions personnalisées.

Pour donner suite à votre demande, le code en Java 17 suivant incorpore les règles et étapes définies dans les images fournies :

Ce code doit être complété par la logique de validation spécifique et la conversion du DTO à l'entité selon les données que vous souhaitez traiter et stocker, en conformité avec les règles de gestion (RG) fournies dans vos captures d'écran.

Le contrôleur gère l'entrée des requêtes HTTP POST pour créer des événements. Le service exécute la logique de validation et la persistance des données. Des exceptions personnalisées sont lancées si les validations échouent.

Veuillez noter que vous devez également gérer les exceptions dans votre contrôleur avec `@ExceptionHandler` pour renvoyer les bonnes réponses HTTP en cas d'erreurs.

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import javax.persistence.*;
import java.sql.Timestamp;

@Entity
@Table(name = "TEVENTSH")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class EventHistoryEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID_T_EVENT_SH", nullable = false)
    private Long idTEventSH;
    
    @Column(name = "ID_TYPE_EVT_Api", length = 50)
    private String idTypeEvtEpi;

    @Column(name = "ID_TYPE_EVT_PSP", length = 50)
    private String idTypeEvtPsp;
    
    @Column(name = "ID_EVT",nullable = false)
    private Integer idEvt;

    @Column(name = "IBAN", length = 34)
    private String iban;

    @Column(name = "STATUS_LIB",length = 2)
    private String statusLib;

    @Column(name = "ERROR_CODE",length = 7)
    private String errorCode;

    @Column(name = "MSG_ERROR",length = 200)
    private String msgError;

    @Column(name = "MSG_EVT")
    private String msgEvt;

    @Column(name = "ID_TECH_EVT",length = 36)
    private String idTechEvt;

    @Column(name = "ID_PRCS",nullable = false)
    private Integer idPrcs;

    @Column(name = "APP_CODE",length = 10)
    private String appCode;

    @Column(name = "ENROL_WAy_LIB",length = 3)
    private String enrolWayLib;

    @Column(name = "ID_USER",length = 10)
    private String idUser;

    @Column(name = "EVENT_DT", nullable = false)
    private Timestamp eventDt;

    // Getters et Setters générés par Lombok...
}

```



```java
// EventRepository.java
@Repository
public interface EventRepository extends JpaRepository<EventHistoryEntity, Long> {
}
```



```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataIntegrityViolationException;

@Service
public class EventService {

    private final EventRepository eventRepository;

    @Autowired
    public EventService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    public Event registerEvent(EventDTO eventDTO) {
        validateEventDTO(eventDTO);
        EventHistoryEntity event = convertToEntity(eventDTO);
        event.setEventDt(Timestamp.valueOf(eventDTO.getEventDt())); // Date actuelle pour l'enregistrement
        try {
            return eventRepository.save(event);
        } catch (DataIntegrityViolationException e) {
            throw new EventRegistrationException("Error during event registration", e);
        }
    }

    private void validateEventDTO(EventDTO eventDTO) {
        // Scénario 1: Tous les paramètres sont valides
        if (Objects.isNull(eventDTO.getIdTypeEvtPsp()) || Objects.isNull(eventDTO.getIban())) {
            // Scénario 2: Paramètre obligatoire absent
            throw new EventValidationException("Paramètre obligatoire manquant.");
        }
        
        // Scénario 2: Format de donnée invalide (ici nous vérifions simplement la longueur de l'IBAN comme exemple)
        if (eventDTO.getIban().length() > 34) {
            // Scénario 3: Format de donnée invalide
            throw new EventValidationException("Format de l'IBAN invalide.");
        }
        // Add other validations as per your requirements
    }

    private EventHistoryEntity convertToEntity(EventDTO eventDTO) {
        // Assuming that EventDTO has the same field names as Event
        return EventHistoryEntity.builder()
                .idTypeEvtPsp(eventDTO.getIdTypeEvtPsp())
                .iban(eventDTO.getIban())
                .idEvt(eventDTO.getIdEvt())
                .statusLib(eventDTO.getStatusLib())
                .errorCode(eventDTO.getErrorCode())
                .msgError(eventDTO.getMsgError())
                .msgEvt(eventDTO.getMsgEvt())
                .idTech(eventDTO.getIdTech())
                .idProcs(eventDTO.getIdProcs())
                .appCode(eventDTO.getAppCode())
                .enrolWalLib(eventDTO.getEnrolWalLib())
                .idUser(eventDTO.getIdUser())
                .build();
    }
}
```



```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import java.util.Map;

@RestController
@RequestMapping("/api/events")
public class EventController {

    private final EventService eventService;

    @Autowired
    public EventController(EventService eventService) {
        this.eventService = eventService;
    }

    @PostMapping
    public ResponseEntity<?> createEvent(@RequestBody Map<String, Object> eventData) {
        try {
            EventDTO eventDTO = convertMapToDTO(eventData);
            EventHistoryEntity event = eventService.registerEvent(eventDTO);
            return ResponseEntity.status(HttpStatus.CREATED).body(event);
        } catch (EventValidationException e) {
            // Gestion des erreurs de validation en fonction des RG
            String message = e.getMessage();
            if (message.contains("RG-0001CE")) {
                return ResponseEntity
                        .status(HttpStatus.BAD_REQUEST)
                        .body(Map.of("error", "Parameter missing", "code", "RG-0001CE"));
            } else if (message.contains("RG-0002CE")) {
                return ResponseEntity
                        .status(HttpStatus.BAD_REQUEST)
                        .body(Map.of("error", "Invalid parameter format", "code", "RG-0002CE"));
            }
            // Autres cas RG si nécessaire
        }
        return ResponseEntity.internalServerError().build(); // En cas d'autres erreurs
    }

    private EventDTO convertMapToDTO(Map<String, Object> eventData) throws EventValidationException {
    try {
        String idTypeEvtEpi = (String) eventData.get("idTypeEvtApi");
        String idTypeEvtPsp = (String) eventData.get("idTypeEvtPsp");
        String iban = (String) eventData.get("iban");
        Integer idEvt = eventData.get("ID_EVT") != null ? Integer.parseInt((String) eventData.get("ID_EVT")) : null;
        String statusLib = (String) eventData.get("STATUS_LIB");
        String errorCode = (String) eventData.get("ERROR_CODE");
        String msgError = (String) eventData.get("MSG_ERROR");
        String msgEvt = (String) eventData.get("MSG_EVT");
        String idTechEvt = (String) eventData.get("ID_TECH_EVT");
        String idProcs = (String) eventData.get("ID_PRCS");
        String appCode = (String) eventData.get("APP_CODE");
        String enrolWayLib = (String) eventData.get("ENROL_WAY_LIB");
        String idUser = (String) eventData.get("ID_USER");
        // Format de date par défaut ou format personnalisé si nécessaire
        DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        LocalDateTime eventDt = eventData.get("EVENT_DT") != null ?
                                LocalDateTime.parse((String) eventData.get("EVENT_DT"), formatter) :
                                LocalDateTime.now(); // Current date and time if not provided
        
        // Validation basique - RG-0001CE
        if (idTypeEvtPsp == null || iban == null) {
            throw new EventValidationException("RG-0001CE: Mandatory parameter is missing.");
        }
        
        // Validation de format - RG-0002CE
        // Exemple pour IBAN
        if (iban.length() > 34) {
            throw new EventValidationException("RG-0002CE: IBAN format is incorrect.");
        }
        
        // Création du DTO en utilisant un constructeur ou un builder
        return new EventDTO(idTypeEvtEpi, idTypeEvtPsp, iban, idEvt,
            statusLib,
            errorCode,
            msgError,
            msgEvt,
            idTechEvt,
            idPrcs,
            appCode,
            enrolWayLib,
            idUser,
            eventDt);
    } catch (NumberFormatException e) {
        throw new EventValidationException("RG-0002CE: Invalid number format.");
    } catch (DateTimeParseException e) {
        throw new EventValidationException("RG-0002CE: Invalid date format.");
    }
}
}
```



```java
// -- Custom Exceptions --
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class EventValidationException extends RuntimeException {
    private final String code;

    public EventValidationException(String message, String code) {
        super(message);
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```



```java
public class EventRegistrationException extends RuntimeException {
    public EventRegistrationException(String message, Throwable cause) {
        super(message, cause);
    }
}
```



```java
// EventDTO.java
import lombok.Data;

@Data
public class EventDTO {
    private String idTypeEvtPsp;
    private String iban;
    private Integer idEvt;
    private String statusLib;
    private String errorCode;
    private String msgError;
    private String msgEvt;
    private String idTech;
    private String idProcs;
    private String appCode;
    private String enrolWayLib;
    private String idUser;
    private LocalDateTime eventDt;

}
```



