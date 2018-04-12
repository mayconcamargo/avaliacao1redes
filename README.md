
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.file.Files;
import java.nio.file.Paths;

public class ServidorWeb {
    final static String DIRETORIO_RAIZ = "c:\\temp\\";

    public static void main (String[] args) throws IOException {
        
        ServerSocket socketServidor = new ServerSocket(8090);
        System.out.println("Servidor iniciado. Loop para conexões iniciado.1");

        while(true) {
            try {
                Socket socket = socketServidor.accept();

                BufferedReader leitura = new BufferedReader(new InputStreamReader(socket.getInputStream()));

                PrintStream escrita = new PrintStream(new BufferedOutputStream(socket.getOutputStream()));

               
                String request = leitura.readLine();
                request = request.substring(5, request.length() - 9).trim();
                if (!request.isEmpty()) {
                    String diretorioArquivo = DIRETORIO_RAIZ + "\\" + request;
                    File arquivo = new File(diretorioArquivo);

                    if (!arquivo.exists() || arquivo.isDirectory()) {
                        String response = "HTTP/1.1 404 Not Found\r\n\r\n<br>Arquivo nao encontrado no servidor e/ou eh um diretorio.";
                        escrita.print(response);
                    } else {
                        OutputStream os = socket.getOutputStream();

                        String header = "HTTP/1.1 200 OK\r\n" +
                                        "Accept-Ranges: bytes\r\n" +
                                        "Content-Type: " + "application/octet-stream" + "\r\n" + 
                                        "Content-Length: "+arquivo.length()+"\r\n" +
                                        "Content-Disposition: attachment; filename=\""+request+"\"\r\n\r\n";

                        os.write(header.getBytes());
                        Files.copy(Paths.get(diretorioArquivo) , os); 
                    }
                } else {
                    String response = "HTTP/1.1 200 OK\r\n\r\n<br>";
                    escrita.print(response);
                }
   
                escrita.close();
                leitura.close();
                socket.close();

            } catch (Exception e) {
                e.printStackTrace(); 
            }
        }
    }
}
