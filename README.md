# JavaGame
public class Main extends Application{
   
   @Override
   
    public void start(Stage primaryStage) throws Exception {
        //GameView view = new GameView();
        StartView view = new StartView();
        Model model = new Model();
        StartPresenter startPresenter = new StartPresenter(model,view);
        Scene scene = new Scene(view);
        scene.getStylesheets().add("css/Style.css");
        primaryStage.setScene(scene);
        primaryStage.setTitle("Educat");
        primaryStage.setHeight(700);
        primaryStage.setWidth(850);
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}

--------------------------------------------------------------------------------------------------------


public class StartView extends VBox {

    private Label nameLbl = new Label("Name");
    private TextField playerName = new TextField();
    private ToggleButton[] choice = new ToggleButton[3];
    private Button begin = new Button("Begin the game!");
    private final ToggleGroup tgGroup = new ToggleGroup();

    public StartView() {
        initialiseNodes();
        layoutnodes();
    }

    private void initialiseNodes() {
        playerName.setEditable(true);
        for (int i = 0; i < 3; i++) {
            this.choice[i] = new ToggleButton("Choice " + (i + 1));
            this.choice[i].setToggleGroup(tgGroup);
        }
    }

    private void layoutnodes() {
        this.setStyle("-fx-background-color: black");
        this.setAlignment(Pos.CENTER);

        HBox hBox = new HBox();
        hBox.getChildren().addAll(nameLbl, playerName);
        hBox.setSpacing(50);
        hBox.setPadding(new Insets(50));
        hBox.setAlignment(Pos.CENTER);


        HBox hBox1 = new HBox();
        hBox1.getChildren().addAll(choice);
        hBox1.setSpacing(10);
        hBox1.setPadding(new Insets(20));
        for (int i = 0; i < 3; i++) {
            choice[i].setPrefSize(100,50);
        }
        hBox1.setAlignment(Pos.CENTER);

        HBox hBox2 = new HBox();
        hBox2.getChildren().add(begin);
        hBox2.setPadding(new Insets(20));
        hBox2.setAlignment(Pos.CENTER);

        this.getChildren().addAll(hBox, hBox1,hBox2);
    }

    public TextField getPlayerName() {
        return playerName;
    }

    public ToggleButton[] getChoice() {
        return choice;
    }

    public ToggleGroup getTgGroup() {
        return tgGroup;
    }

    public Button getBegin() {
        return begin;
    }
}

----------------------------------------------------------------------------------------------------------

public class StartPresenter {

    private Model model;
    private StartView startView;
    private Speler speler;

    public StartPresenter(Model model, StartView startView) {
        this.model = new Model();
        this.startView = startView;
        addEventHandlers();
    }

    private void addEventHandlers(){
        startView.getBegin().setOnAction(new EventHandler<ActionEvent>() {
            @Override
            public void handle(ActionEvent event) {
                GameView gameView = new GameView();
                GamePresenter gamePresenter = new GamePresenter(gameView,model);
                startView.getScene().setRoot(gameView);
                gameView.getScene().getWindow().sizeToScene();
                speler.naam = (startView.getPlayerName().getText());
            }
        });
    }
}

---------------------------------------------------------------------------------------------------------------

public class GameView extends BorderPane {

    private Model model;
    private Button[] statement = new Button[7];
    private Label lbl = new Label();
    private Label scoreLbl = new Label();
    private Label timeLbl = new Label();
    private Label playerName = new Label();
    private Speler speler;

    public GameView() {
        this.initialiseNodes();
        this.layoutNodes();
    }

    private void initialiseNodes() {
        this.model = new Model();
        model.leesBestand();
        for (int i = 0; i < 7; i++) {
            this.statement[i] = new Button(model.getPossibleSolutions().get(i));
        }

        this.lbl = new Label(model.getQuestionAsked().get(model.getCurrentQuestionIndex()));
        this.playerName = new Label(speler.naam);
        this.scoreLbl = new Label("Score!");
        this.timeLbl = new Label("Time!");
    }

    private void layoutNodes() {
        VBox vbox = new VBox();
        vbox.setPadding(new Insets(20));
        vbox.setSpacing(10);
        vbox.getChildren().addAll(statement);
        vbox.setStyle("-fx-background-color: black");
        this.setCenter(vbox);

        HBox hbox = new HBox();
        hbox.setPadding(new Insets(20));
        hbox.setStyle("-fx-background-color: darkgoldenrod");
        hbox.setSpacing(10);
        hbox.getChildren().add(playerName);
        hbox.getChildren().add(scoreLbl);
        hbox.getChildren().add(timeLbl);
        playerName.setPrefSize(100,40);
        scoreLbl.setPrefSize(100,40);
        timeLbl.setPrefSize(100,40);
        playerName.setAlignment(Pos.CENTER);
        scoreLbl.setAlignment(Pos.CENTER);
        timeLbl.setAlignment(Pos.CENTER);
        hbox.setAlignment(Pos.BASELINE_CENTER);
        this.setTop(hbox);

        HBox hbox1 = new HBox();
        hbox1.setPadding(new Insets(20));
        hbox1.setStyle("-fx-background-color: darkgoldenrod");
        hbox1.setSpacing(100);
        hbox1.getChildren().add(lbl);
        lbl.setPrefSize(100,40);
        lbl.setAlignment(Pos.CENTER);
        hbox1.setAlignment(Pos.BASELINE_CENTER);
        this.setBottom(hbox1);
    }

    Button[] getStatement() {
        return statement;
    }

    Label getLbl() {
        return lbl;
    }
}

------------------------------------------------------------------------------------------------------------------

public class GamePresenter {

    private GameView view;
    private Model model;

    public GamePresenter(GameView view, Model model) {
        this.view = view;
        this.model = model;
        addEventHandlers();
        updateView();
    }

    private void addEventHandlers(){
        Button[] statements = view.getStatement();
        for (int i = 0; i < statements.length; i++) {
            String answer = statements[i].getText();
            statements[i].setOnMouseClicked(new EventHandler<MouseEvent>() {
                @Override
                public void handle(MouseEvent event) {
                    model.checkValidAnswer(answer);
                    updateView();
                }
            });
        }
    }

    private void updateView() {
        //Nieuwe question, score toevoegen en tijd toevoegen
        view.getLbl().setText(model.getQuestionAsked().get(model.getCurrentQuestionIndex()));
    }
}

---------------------------------------------------------------------------------------------------------------------

public class Model {

    private List<String> questionAsked = new ArrayList<>();
    private List<String> possibleSolutions = new ArrayList<>();
    private int currentQuestionIndex = 0;

    public void leesBestand() {
        String[] ss = new String[15];

        try (BufferedReader reader = new BufferedReader(new FileReader("src/tekstbestanden/Opgave1.txt"))) {
            String line = null;
            while ((line = reader.readLine()) != null) {
                line = line.replaceAll("\t","\n");
                ss = line.split("\n");
                for (int i = 0; i < ss.length ; i++) {
                    if((i%2)==0){
                        questionAsked.add(ss[i]);
                    } else{
                        possibleSolutions.add(ss[i]);
                    }
                }
            }
        } catch (IOException ex) {
            System.out.println("No file");
        }
    }


    public void checkValidAnswer(String answer) {
        if (possibleSolutions.get(currentQuestionIndex).equals(answer)) {
            System.out.println("juist");
            //goed geluid afspelen
            Media correctSound = new Media(new File("src/geluidbestanden/CorrectSoundEffect.mp3").toURI().toString());
            MediaPlayer mediaPlayer = new MediaPlayer(correctSound);
            mediaPlayer.play();
        } else {
            System.out.println("fout");
            //fout geluid afspelen
            Media incorrectSound = new Media(new File("src/geluidbestanden/IncorrectSoundEffect.mp3").toURI().toString());
            MediaPlayer mediaPlayer = new MediaPlayer(incorrectSound);
            mediaPlayer.play();
        }
        currentQuestionIndex++;

    }

    public List<String> getQuestionAsked() {
        return this.questionAsked;
    }

    public List<String> getPossibleSolutions() {
        return this.possibleSolutions;
    }

    public int getCurrentQuestionIndex() {
        return currentQuestionIndex;
    }
}

----------------------------------------------------------------------------------------------------------------------------

public class Speler {

    public String naam;

    public Speler(String naam) {
        this.naam = naam;
    }
}
