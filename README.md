# WordsFromWord
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class WordsFromWord {
    record Config(int minSubWordLength, int minWordsToPass,
                  int easyMaxMainWordLength, int mediumMaxMainWordLength,
                  int easyMaxSubWords, int mediumMaxSubWords) {

        /** Values from the assignment's example config. */
        static Config defaults() {
            return new Config(3, 8, 7, 10, 15, 30);
        }

        /** Reads config.json (format from the assignment). */
        static Config fromFile(Path path) throws Exception {
            String json = Files.readString(path, StandardCharsets.UTF_8);
            return new Config(
                    num(json, "minSubWordLength"),
                    num(json, "minWordsToPass"),
                    num(json, "easyMaxMainWordLength"),
                    num(json, "mediumMaxMainWordLength"),
                    num(json, "easyMaxSubWords"),
                    num(json, "mediumMaxSubWords"));
        }

        private static int num(String json, String key) {
            Matcher m = Pattern.compile("\"" + key + "\"\\s*:\\s*(\\d+)").matcher(json);
            if (!m.find()) throw new IllegalArgumentException("config.json: field \"" + key + "\" not found");
            return Integer.parseInt(m.group(1));
        }
    }

    record Level(int id, String mainWord, String difficulty, int minWordsToPass, List<String> subWords) {}

    static final int ALPHA = 32; // А..Я (Ё is folded into Е)

    /** Upper case, Ё -> Е. */
    static String norm(String s) {
        return s.trim().toUpperCase(Locale.ROOT).replace('Ё', 'Е');
    }

    /** Letter counts, or null if the word has non-Russian characters. */
    static int[] counts(String w) {
        int[] c = new int[ALPHA];
        for (int i = 0; i < w.length(); i++) {
            int k = w.charAt(i) - 'А';
            if (k < 0 || k >= ALPHA) return null;
            c[k]++;
        }
        return c;
    }

    static List<Level> generate(Collection<String> dictionary, Collection<String> mainWords, Config cfg) {
        // Dedupe by normalized form once; precompute letter counts once (performance).
        Map<String, String> display = new LinkedHashMap<>(); // norm -> uppercase original (keeps Ё)
        for (String raw : dictionary) {
            String n = norm(raw);
            if (n.length() >= cfg.minSubWordLength() && counts(n) != null)
                display.putIfAbsent(n, raw.trim().toUpperCase(Locale.ROOT));
        }
        List<String> keys = new ArrayList<>(display.keySet());
        List<int[]> keyCounts = new ArrayList<>(keys.size());
        for (String k : keys) keyCounts.add(counts(k));

        List<Level> levels = new ArrayList<>();
        int id = 1;
        for (String rawMain : mainWords) {
            String main = norm(rawMain);
            int[] mc = counts(main);
            if (mc == null || main.isEmpty()) {
                System.err.println("Skipped (bad main word): " + rawMain);
                continue;
            }
            List<String> subs = new ArrayList<>();
            for (int i = 0; i < keys.size(); i++) {
                String k = keys.get(i);
                if (k.length() > main.length() || k.equals(main)) continue;
                int[] kc = keyCounts.get(i);
                boolean ok = true;
                for (int j = 0; j < ALPHA; j++) {
                    if (kc[j] > mc[j]) { ok = false; break; }
                }
                if (ok) subs.add(display.get(k));
            }
            if (subs.size() < cfg.minWordsToPass()) {
                System.err.println("Skipped (" + subs.size() + " < " + cfg.minWordsToPass() + "): " + rawMain);
                continue;
            }
            String diff;
            if (main.length() <= cfg.easyMaxMainWordLength() && subs.size() <= cfg.easyMaxSubWords()) diff = "EASY";
            else if (main.length() <= cfg.mediumMaxMainWordLength() && subs.size() <= cfg.mediumMaxSubWords()) diff = "MEDIUM";
            else diff = "HARD";
            levels.add(new Level(id++, rawMain.trim().toUpperCase(Locale.ROOT), diff, cfg.minWordsToPass(), subs));
        }
        return levels;
    }

    static String esc(String s) {
        return s.replace("\\", "\\\\").replace("\"", "\\\"");
    }

    static String toJson(List<Level> levels) {
        StringBuilder sb = new StringBuilder("{\n  \"levels\": [\n");
        for (int i = 0; i < levels.size(); i++) {
            Level l = levels.get(i);
            sb.append("    {\n")
                    .append("      \"id\": ").append(l.id()).append(",\n")
                    .append("      \"mainWord\": \"").append(esc(l.mainWord())).append("\",\n")
                    .append("      \"difficulty\": \"").append(l.difficulty()).append("\",\n")
                    .append("      \"minWordsToPass\": ").append(l.minWordsToPass()).append(",\n")
                    .append("      \"subWords\": [\n");
            for (int j = 0; j < l.subWords().size(); j++) {
                sb.append("        { \"text\": \"").append(esc(l.subWords().get(j))).append("\", \"hint\": \"\" }")
                        .append(j + 1 < l.subWords().size() ? ",\n" : "\n");
            }
            sb.append("      ]\n    }").append(i + 1 < levels.size() ? ",\n" : "\n");
        }
        return sb.append("  ]\n}").toString();
    }

    /** Reads a UTF-8 file, one word per line; skips empty lines and a BOM. */
    static List<String> readWords(Path path) throws Exception {
        List<String> words = new ArrayList<>();
        for (String line : Files.readAllLines(path, StandardCharsets.UTF_8)) {
            String w = line.replace("﻿", "").trim();
            if (!w.isEmpty()) words.add(w);
        }
        return words;
    }

    public static void main(String[] args) {
        if (args.length < 2) {
            System.err.println("Usage: java WordsFromWord dict.txt mains.txt [config.json]");
            System.exit(1);
        }
        try {
            List<String> dict = readWords(Path.of(args[0]));
            List<String> mains = readWords(Path.of(args[1]));
            Config cfg = args.length >= 3 ? Config.fromFile(Path.of(args[2])) : Config.defaults();

            String json = toJson(generate(dict, mains, cfg));
            System.out.println(json);
            Files.writeString(Path.of("levels.json"), json, StandardCharsets.UTF_8);
        } catch (java.nio.file.NoSuchFileException e) {
            System.err.println("File not found: " + e.getFile());
            System.exit(1);
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
            System.exit(1);
        }
    }
}
