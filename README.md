# JavaGame
public class Model {

    private List<String> questionAsked = new ArrayList<>();
    private List<String> possibleSolutions = new ArrayList<>();
    private boolean finished;
    private boolean correct;
    private int currentQuestionIndex = 0;
    private int score = 0;
    private int minutes = 0;
    private int seconds = 0;
    private int tickDurationMillis = 1000;
    private int filechosen;

    public void leesBestand() {
        String[] ss = new String[26];

        try (BufferedReader reader = new BufferedReader(new FileReader("src/tekstbestanden/" + filechosen + ".txt"))) {
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
            score++;
            correct = true;
            //goed geluid afspelen
            Media correctSound = new Media(new File("src/geluidbestanden/CorrectSoundEffect.mp3").toURI().toString());
            MediaPlayer mediaPlayer = new MediaPlayer(correctSound);
            mediaPlayer.play();
        } else {
            System.out.println("fout");
            correct = false;
            //fout geluid afspelen
            Media incorrectSound = new Media(new File("src/geluidbestanden/IncorrectSoundEffect.mp3").toURI().toString());
            MediaPlayer mediaPlayer = new MediaPlayer(incorrectSound);
            mediaPlayer.play();
        }
        currentQuestionIndex++;

        if(currentQuestionIndex == possibleSolutions.size()){
            finished = true;
        } else{
            finished = false;
        }

    }

    public void tick(){
        this.seconds++;

        if(minutes==2){
            finished = true;
        }

        if (this.seconds == 60){
            this.seconds = 0;
            this.minutes++;
        }
    }

    public boolean isCorrect() {
        return correct;
    }

    public boolean gameFinished(){
        return finished;
    }

    public int getScore() {
        return score;
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

    public int getMinutes() {
        return minutes;
    }

    public int getSeconds() {
        return seconds;
    }

    public int getTickDurationMillis() {
        return tickDurationMillis;
    }

    public void setFilechosen(int filechosen) {
        this.filechosen = filechosen;
    }

}

-----------------------------------------------------------------------------------------------------------------------

startView.getTgGroup().selectedToggleProperty().addListener(new ChangeListener<Toggle>() {
            @Override
            public void changed(ObservableValue<? extends Toggle> observable, Toggle oldValue, Toggle newValue) {
                if (newValue != null) {
                    int filechosen = (int) startView.getTgGroup().getSelectedToggle().getUserData();
                    model.setFilechosen(filechosen);
                }
            }
        });
