import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.*;
import javax.swing.*;

class Question {
    String category;
    String questionText;
    String[] options;
    int correctAnswer;

    public Question(String category, String questionText, String[] options, int correctAnswer) {
        this.category = category;
        this.questionText = questionText;
        this.options = options;
        this.correctAnswer = correctAnswer;
    }

    public boolean isCorrect(int answer) {
        return answer == correctAnswer;
    }
}

public class QuizApplication extends Frame implements ActionListener {
    ArrayList<Question> questions = new ArrayList<>();
    ArrayList<Question> selectedQuestions = new ArrayList<>();
    int currentQuestionIndex = 0;
    int score = 0;

    Label questionLabel;
    CheckboxGroup optionsGroup;
    Checkbox option1, option2, option3, option4;
    Button nextButton;
    Label scoreLabel;
    String selectedCategory;

    public QuizApplication() {
        setLayout(new FlowLayout());

        questionLabel = new Label("", Label.CENTER);
        add(questionLabel);

        optionsGroup = new CheckboxGroup();
        option1 = new Checkbox("", optionsGroup, false);
        option2 = new Checkbox("", optionsGroup, false);
        option3 = new Checkbox("", optionsGroup, false);
        option4 = new Checkbox("", optionsGroup, false);

        add(option1);
        add(option2);
        add(option3);
        add(option4);

        nextButton = new Button("Next");
        nextButton.addActionListener(this);
        add(nextButton);

        scoreLabel = new Label("");
        add(scoreLabel);

        setTitle("Quiz Application");
        setSize(400, 300);
        setVisible(true);
    }

    public void loadQuestions(String fileName, String category) {
        try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split("\\|");
                if (parts.length != 7) continue;

                String fileCategory = parts[0];
                String questionText = parts[1];
                String[] options = Arrays.copyOfRange(parts, 2, 6);
                int correctAnswer = Integer.parseInt(parts[6]);

                if (fileCategory.equalsIgnoreCase(category)) {
                    questions.add(new Question(fileCategory, questionText, options, correctAnswer));
                }
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
    }

    public void startQuiz(String category) {
        selectedCategory = category;

        // Shuffle and pick 5 questions
        Collections.shuffle(questions);
        selectedQuestions = new ArrayList<>(questions.subList(0, Math.min(5, questions.size())));

        loadQuestion();
    }

    private void loadQuestion() {
        if (currentQuestionIndex >= selectedQuestions.size()) {
            displayScore();
            return;
        }

        Question currentQuestion = selectedQuestions.get(currentQuestionIndex);

        questionLabel.setText(currentQuestion.questionText);
        option1.setLabel(currentQuestion.options[0]);
        option2.setLabel(currentQuestion.options[1]);
        option3.setLabel(currentQuestion.options[2]);
        option4.setLabel(currentQuestion.options[3]);
    }

    private void displayScore() {
        questionLabel.setText("Quiz Completed!");
        option1.setVisible(false);
        option2.setVisible(false);
        option3.setVisible(false);
        option4.setVisible(false);
        nextButton.setVisible(false);

        scoreLabel.setText("Your score: " + score + "/" + selectedQuestions.size());
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (currentQuestionIndex < selectedQuestions.size()) {
            Question currentQuestion = selectedQuestions.get(currentQuestionIndex);

            // Check the selected answer
            int selectedOption = -1;
            if (option1.getState()) selectedOption = 1;
            if (option2.getState()) selectedOption = 2;
            if (option3.getState()) selectedOption = 3;
            if (option4.getState()) selectedOption = 4;

            if (currentQuestion.isCorrect(selectedOption)) {
                score++;
            }

            currentQuestionIndex++;
            loadQuestion();
        }
    }

    public static void main(String[] args) {
        QuizApplication quizApp = new QuizApplication();

        // Choose a category
        String[] categories = {"General Knowledge", "Maths", "Games", "Movies", "Bikes"};
        String selectedCategory = (String) JOptionPane.showInputDialog(
                null,
                "Choose a category:",
                "Quiz Categories",
                JOptionPane.QUESTION_MESSAGE,
                null,
                categories,
                categories[0]);

        if (selectedCategory == null || selectedCategory.isEmpty()) {
            System.exit(0);
        }

        quizApp.loadQuestions("questions.txt", selectedCategory);
        quizApp.startQuiz(selectedCategory);
    }
}
