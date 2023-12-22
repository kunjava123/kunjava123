import org.telegram.telegrambots.bots.TelegramLongPollingBot;
import org.telegram.telegrambots.meta.api.methods.send.SendMessage;
import org.telegram.telegrambots.meta.api.objects.Update;
import org.telegram.telegrambots.meta.exceptions.TelegramApiException;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class YouTubeDownloaderBot extends TelegramLongPollingBot {

    private static final String YOUR_BOT_TOKEN = "YOUR_BOT_TOKEN";

    @Override
    public String getBotToken() {
        return YOUR_BOT_TOKEN;
    }

    @Override
    public void onUpdateReceived(Update update) {
        if (update.hasMessage() && update.getMessage().hasText()) {
            String messageText = update.getMessage().getText();
            long chatId = update.getMessage().getChatId();

            if (messageText.startsWith("/download")) {
                String videoId = messageText.substring("/download".length());
                String command = String.format("youtube-dl -o \"%%(title)s.%%(ext)s\" https://www.youtube.com/watch?v=%s", videoId);

                try {
                    Process process = Runtime.getRuntime().exec(command);
                    int exitCode = process.waitFor();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));

                    StringBuilder output = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        output.append(line).append("\n");
                    }

                    String response = (exitCode == 0) ? "Download successful!" : "Download failed.";

                    SendMessage message = new SendMessage()
                            .setChatId(chatId)
                            .setText(response + "\n\n" + output.toString());
                    execute(message);
                } catch (IOException | InterruptedException | TelegramApiException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Override
    public String getBotUsername() {
        return "YourBotUsername";
    }

    public static void main(String[] args) {
        TelegramBotsApi botsApi = new TelegramBotsApi();

        try {
            botsApi.registerBot(new YouTubeDownloaderBot());
        } catch (TelegramApiException e) {
            e.printStackTrace();
        }
    }
}
