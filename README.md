# java-meeting-summarizer
java-meeting-summarizer....../......
import java.util.*;

public class MeetingSummarizer {

    static final Set<String> STOP_WORDS = new HashSet<>(Arrays.asList(
            "the", "is", "a", "an", "and", "to", "of", "in", "for", "on", "with", "we", "will", "that", "this", "it", "be"
    ));

    static final List<String> ACTION_KEYWORDS = Arrays.asList(
            "will", "must", "need to", "should", "assigned", "deadline", "by friday", "by monday", "follow up"
    );

    // Score each sentence by frequency of important words
    static Map<String, Integer> wordFrequency(String text) {
        Map<String, Integer> freq = new HashMap<>();
        for (String w : text.toLowerCase().split("\\W+")) {
            if (w.length() > 2 && !STOP_WORDS.contains(w)) {
                freq.put(w, freq.getOrDefault(w, 0) + 1);
            }
        }
        return freq;
    }

    static List<String> summarize(String notes, int topN) {
        String[] sentences = notes.split("(?<=[.!?])\\s+");
        Map<String, Integer> freq = wordFrequency(notes);
        Map<String, Integer> sentenceScores = new LinkedHashMap<>();

        for (String s : sentences) {
            int score = 0;
            for (String w : s.toLowerCase().split("\\W+")) {
                score += freq.getOrDefault(w, 0);
            }
            sentenceScores.put(s, score);
        }

        List<Map.Entry<String, Integer>> sorted = new ArrayList<>(sentenceScores.entrySet());
        sorted.sort((a, b) -> b.getValue() - a.getValue());

        List<String> top = new ArrayList<>();
        for (int i = 0; i < Math.min(topN, sorted.size()); i++) {
            top.add(sorted.get(i).getKey());
        }
        return top;
    }

    static List<String> extractActionItems(String notes) {
        List<String> actions = new ArrayList<>();
        for (String s : notes.split("(?<=[.!?])\\s+")) {
            String lower = s.toLowerCase();
            for (String kw : ACTION_KEYWORDS) {
                if (lower.contains(kw)) {
                    actions.add(s.trim());
                    break;
                }
            }
        }
        return actions;
    }

    public static void main(String[] args) {
        String notes = "The team discussed the new project launch timeline. "
                + "Rahul will prepare the project report by Friday. "
                + "The client wants an updated project demo next week. "
                + "Priya must follow up with the design team about the launch assets. "
                + "Everyone agreed that the project deadline is critical. "
                + "The meeting ended with a plan to review progress on Monday.";

        System.out.println("=== Meeting Summary ===");
        for (String s : summarize(notes, 3)) {
            System.out.println("- " + s);
        }

        System.out.println("\n=== Action Items ===");
        for (String a : extractActionItems(notes)) {
            System.out.println("* " + a);
        }
    }
}
