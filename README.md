import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.OffsetDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Scanner;

import org.json.JSONObject;

public class FlightPredictor {

    private static final String API_KEY = "YOUR_API_KEY"; // Replace with your Aviation Edge API key
    private static final String BASE_URL = "https://api.aviationedge.com/v2/public/flights";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter flight number: ");
        String flightNumber = scanner.nextLine();

        System.out.print("Enter airline code: ");
        String airlineCode = scanner.nextLine();

        System.out.print("Enter date (YYYY-MM-DD): ");
        String date = scanner.nextLine();

        scanner.close();

        FlightData flightData = getFlightData(flightNumber, airlineCode, date);

        if (flightData != null) {
            System.out.println("Predicted Takeoff Time: " + flightData.getTakeoffTime());
            System.out.println("Predicted Arrival Time: " + flightData.getArrivalTime());
        } else {
            System.out.println("No flight data found.");
        }
    }

    private static FlightData getFlightData(String flightNumber, String airlineCode, String date) {
        try {
            String url = String.format("%s/%s/%s?key=%s&date=%s", BASE_URL, airlineCode, flightNumber, API_KEY, date);
            URI uri = URI.create(url);

            HttpRequest request = HttpRequest.newBuilder(uri)
                    .GET()
                    .build();

            HttpClient client = HttpClient.newHttpClient();
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() == 200) {
                JSONObject jsonData = new JSONObject(response.body());
                return new FlightData(jsonData.getString("departure_scheduled"), jsonData.getString("arrival_scheduled"));
            } else {
                System.out.println("Error retrieving data: " + response.statusCode());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    static class FlightData {
        private OffsetDateTime takeoffTime;
        private OffsetDateTime arrivalTime;

        public FlightData(String takeoffTimeStr, String arrivalTimeStr) {
            DateTimeFormatter formatter = DateTimeFormatter.ISO_OFFSET_DATE_TIME;
            this.takeoffTime = OffsetDateTime.parse(takeoffTimeStr, formatter);
            this.arrivalTime = OffsetDateTime.parse(arrivalTimeStr, formatter);
        }

        public OffsetDateTime getTakeoffTime() {
            return takeoffTime;
        }

        public OffsetDateTime getArrivalTime() {
            return arrivalTime;
        }
    }
}

