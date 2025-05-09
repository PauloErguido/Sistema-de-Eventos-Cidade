# Sistema-de-Eventos-Cidade
// SistemaEventosCidade.java
// Sistema de cadastro e notificação de eventos em console

import java.io.*;
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.*;

class Usuario {
    private String nome;
    private String email;
    private String cidade;
    private List<String> eventosConfirmados;

    public Usuario(String nome, String email, String cidade) {
        this.nome = nome;
        this.email = email;
        this.cidade = cidade;
        this.eventosConfirmados = new ArrayList<>();
    }

    public String getNome() { return nome; }
    public String getEmail() { return email; }
    public List<String> getEventosConfirmados() { return eventosConfirmados; }

    public void confirmarPresenca(String evento) {
        if (!eventosConfirmados.contains(evento)) {
            eventosConfirmados.add(evento);
        }
    }

    public void cancelarPresenca(String evento) {
        eventosConfirmados.remove(evento);
    }
}

class Evento implements Serializable {
    private String nome;
    private String endereco;
    private String categoria;
    private LocalDateTime horario;
    private String descricao;
    private List<String> participantes;

    public Evento(String nome, String endereco, String categoria, LocalDateTime horario, String descricao) {
        this.nome = nome;
        this.endereco = endereco;
        this.categoria = categoria;
        this.horario = horario;
        this.descricao = descricao;
        this.participantes = new ArrayList<>();
    }

    public String getNome() { return nome; }
    public LocalDateTime getHorario() { return horario; }
    public String getDescricao() { return descricao; }
    public String getEndereco() { return endereco; }
    public String getCategoria() { return categoria; }
    public List<String> getParticipantes() { return participantes; }

    public boolean estaOcorrendo() {
        LocalDateTime agora = LocalDateTime.now();
        return horario.isBefore(agora.plusHours(1)) && horario.isAfter(agora.minusHours(1));
    }

    public boolean jaOcorreu() {
        return horario.isBefore(LocalDateTime.now());
    }

    public void adicionarParticipante(String email) {
        if (!participantes.contains(email)) participantes.add(email);
    }

    public void removerParticipante(String email) {
        participantes.remove(email);
    }

    public String toString() {
        return String.format("%s - %s\nCategoria: %s\nHorário: %s\n%s\n", nome, endereco, categoria,
                horario.format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")), descricao);
    }
}

class EventoManager {
    private List<Evento> eventos;
    private final String FILE_NAME = "events.data";

    public EventoManager() {
        eventos = new ArrayList<>();
        carregarEventos();
    }

    public void adicionarEvento(Evento evento) {
        eventos.add(evento);
        salvarEventos();
    }

    public List<Evento> getEventosOrdenados() {
        eventos.sort(Comparator.comparing(Evento::getHorario));
        return eventos;
    }

    public void salvarEventos() {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(FILE_NAME))) {
            out.writeObject(eventos);
        } catch (IOException e) {
            System.out.println("Erro ao salvar eventos: " + e.getMessage());
        }
    }

    public void carregarEventos() {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(FILE_NAME))) {
            eventos = (List<Evento>) in.readObject();
        } catch (Exception e) {
            eventos = new ArrayList<>();
        }
    }

    public List<Evento> getEventos() { return eventos; }
    public Evento buscarEventoPorNome(String nome) {
        for (Evento e : eventos) {
            if (e.getNome().equalsIgnoreCase(nome)) return e;
        }
        return null;
    }
}

public class SistemaEventosCidade {
    private static Scanner scanner = new Scanner(System.in);
    private static EventoManager eventoManager = new EventoManager();
    private static Usuario usuario;

    public static void main(String[] args) {
        System.out.println("Bem-vindo ao sistema de eventos da cidade!");
        criarUsuario();
        int opcao;
        do {
            exibirMenu();
            opcao = Integer.parseInt(scanner.nextLine());
            switch (opcao) {
                case 1 -> cadastrarEvento();
                case 2 -> listarEventos();
                case 3 -> confirmarPresenca();
                case 4 -> cancelarPresenca();
                case 5 -> meusEventos();
                case 0 -> System.out.println("Saindo...");
                default -> System.out.println("Opção inválida!");
            }
        } while (opcao != 0);
    }

    private static void criarUsuario() {
        System.out.print("Digite seu nome: ");
        String nome = scanner.nextLine();
        System.out.print("Digite seu email: ");
        String email = scanner.nextLine();
        System.out.print("Digite sua cidade: ");
        String cidade = scanner.nextLine();
        usuario = new Usuario(nome, email, cidade);
    }

    private static void exibirMenu() {
        System.out.println("\nMenu:");
        System.out.println("1. Cadastrar evento");
        System.out.println("2. Listar eventos");
        System.out.println("3. Confirmar presença");
        System.out.println("4. Cancelar presença");
        System.out.println("5. Meus eventos confirmados");
        System.out.println("0. Sair");
        System.out.print("Escolha uma opção: ");
    }

    private static void cadastrarEvento() {
        System.out.print("Nome do evento: ");
        String nome = scanner.nextLine();
        System.out.print("Endereço: ");
        String endereco = scanner.nextLine();
        System.out.print("Categoria (Festa, Show, Esporte, etc): ");
        String categoria = scanner.nextLine();
        System.out.print("Data e hora (dd/MM/yyyy HH:mm): ");
        String dataStr = scanner.nextLine();
        System.out.print("Descrição: ");
        String descricao = scanner.nextLine();

        LocalDateTime data = LocalDateTime.parse(dataStr, DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm"));
        Evento evento = new Evento(nome, endereco, categoria, data, descricao);
        eventoManager.adicionarEvento(evento);
        System.out.println("Evento cadastrado!");
    }

    private static void listarEventos() {
        List<Evento> eventos = eventoManager.getEventosOrdenados();
        for (Evento e : eventos) {
            System.out.println(e);
            System.out.println("Status: " + (e.estaOcorrendo() ? "Ocorrendo agora" : e.jaOcorreu() ? "Já ocorreu" : "Por vir"));
        }
    }

    private static void confirmarPresenca() {
        System.out.print("Digite o nome do evento: ");
        String nome = scanner.nextLine();
        Evento evento = eventoManager.buscarEventoPorNome(nome);
        if (evento != null) {
            evento.adicionarParticipante(usuario.getEmail());
            usuario.confirmarPresenca(nome);
            eventoManager.salvarEventos();
            System.out.println("Presença confirmada!");
        } else {
            System.out.println("Evento não encontrado.");
        }
    }

    private static void cancelarPresenca() {
        System.out.print("Digite o nome do evento: ");
        String nome = scanner.nextLine();
        Evento evento = eventoManager.buscarEventoPorNome(nome);
        if (evento != null) {
            evento.removerParticipante(usuario.getEmail());
            usuario.cancelarPresenca(nome);
            eventoManager.salvarEventos();
            System.out.println("Presença cancelada!");
        } else {
            System.out.println("Evento não encontrado.");
        }
    }

    private static void meusEventos() {
        for (String nomeEvento : usuario.getEventosConfirmados()) {
            Evento evento = eventoManager.buscarEventoPorNome(nomeEvento);
            if (evento != null) System.out.println(evento);
        }
    }
}
