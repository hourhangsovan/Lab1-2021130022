file:///D:/大学三年/大三春/软件工程-Software%20Engineering/实验/LAB1/Shiyan1/Lab1.java
### java.util.NoSuchElementException: next on empty iterator

occurred in the presentation compiler.

presentation compiler configuration:
Scala version: 3.3.3
Classpath:
<HOME>\AppData\Local\Coursier\cache\v1\https\repo1.maven.org\maven2\org\scala-lang\scala3-library_3\3.3.3\scala3-library_3-3.3.3.jar [exists ], <HOME>\AppData\Local\Coursier\cache\v1\https\repo1.maven.org\maven2\org\scala-lang\scala-library\2.13.12\scala-library-2.13.12.jar [exists ]
Options:



action parameters:
offset: 4105
uri: file:///D:/大学三年/大三春/软件工程-Software%20Engineering/实验/LAB1/Shiyan1/Lab1.java
text:
```scala
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.HashMap;
import java.util.HashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Random;
import java.util.Scanner;
import java.util.Set;

public class Lab1 {
    private static Map<String, Map<String, Integer>> graph = new HashMap<>();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter the file path: ");
        String filePath = scanner.nextLine();
        try {
            readFile(filePath);
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
            return;
        }

        while (true) {
            System.out.println("Choose an option:");
            System.out.println("1. Show Directed Graph");
            System.out.println("2. Query Bridge Words");
            System.out.println("3. Generate New Text");
            System.out.println("4. Calculate Shortest Path");
            System.out.println("5. Random Walk");
            System.out.println("6. Exit");

            int choice = scanner.nextInt();
            scanner.nextLine();  // consume newline

            switch (choice) {
                case 1:
                    showDirectedGraph();
                    break;
                case 2:
                    System.out.print("Enter the first word: ");
                    String word1 = scanner.nextLine();
                    System.out.print("Enter the second word: ");
                    String word2 = scanner.nextLine();
                    System.out.println(queryBridgeWords(word1, word2));
                    break;
                case 3:
                    System.out.print("Enter the new text: ");
                    String inputText = scanner.nextLine();
                    System.out.println(generateNewText(inputText));
                    break;
                case 4:
                    System.out.print("Enter the first word: ");
                    String start = scanner.nextLine();
                    System.out.print("Enter the second word: ");
                    String end = scanner.nextLine();
                    System.out.println(calcShortestPath(start, end));
                    break;
                case 5:
                    System.out.println(randomWalk());
                    break;
                case 6:
                    System.exit(0);
                default:
                    System.out.println("Invalid choice. Try again.");
            }
        }
    }

    private static void readFile(String filePath) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(filePath));
        String line;
        String prevWord = null;

        while ((line = reader.readLine()) != null) {
            line = line.toLowerCase().replaceAll("[^a-zA-Z\\s]", " ");
            String[] words = line.split("\\s+");

            for (String word : words) {
                if (word.isEmpty()) continue;
                if (prevWord != null) {
                    graph.computeIfAbsent(prevWord, k -> new HashMap<>()).merge(word, 1, Integer::sum);
                }
                prevWord = word;
            }
        }
        reader.close();
    }

    private static void showDirectedGraph() {
        System.out.println("Directed Graph:");
        for (String src : graph.keySet()) {
            for (String dest : graph.get(src).keySet()) {
                System.out.println(src + " -> " + dest + " (weight: " + graph.get(src).get(dest) + ")");
            }
        }
    }

    private static String queryBridgeWords(String word1, String word2) {
        word1 = word1.toLowerCase();
        word2 = word2.toLowerCase();

        if (!graph.containsKey(word1) || !graph.containsKey(word2)) {
            return "No " + "word@@1 + " or " + word2 + " in the graph!";
        }

        List<String> bridgeWords = new ArrayList<>();
        for (String bridge : graph.get(word1).keySet()) {
            if (graph.get(bridge).containsKey(word2)) {
                bridgeWords.add(bridge);
            }
        }

        if (bridgeWords.isEmpty()) {
            return "No bridge words from " + word1 + " to " + word2 + "!";
        } else {
            return "The bridge words from " + word1 + " to " + word2 + " are: " + String.join(", ", bridgeWords) + ".";
        }
    }

    private static String generateNewText(String inputText) {
        String[] words = inputText.toLowerCase().split("\\s+");
        StringBuilder newText = new StringBuilder();

        for (int i = 0; i < words.length - 1; i++) {
            newText.append(words[i]).append(" ");
            List<String> bridgeWords = new ArrayList<>();
            if (graph.containsKey(words[i])) {
                for (String bridge : graph.get(words[i]).keySet()) {
                    if (graph.get(bridge).containsKey(words[i + 1])) {
                        bridgeWords.add(bridge);
                    }
                }
            }

            if (!bridgeWords.isEmpty()) {
                String bridgeWord = bridgeWords.get(new Random().nextInt(bridgeWords.size()));
                newText.append(bridgeWord).append(" ");
            }
        }
        newText.append(words[words.length - 1]);

        return newText.toString();
    }

    private static String calcShortestPath(String word1, String word2) {
        word1 = word1.toLowerCase();
        word2 = word2.toLowerCase();

        if (!graph.containsKey(word1)) return "No " + word1 + " in the graph!";
        if (!graph.containsKey(word2)) return "No " + word2 + " in the graph!";

        Map<String, Integer> distances = new HashMap<>();
        Map<String, String> predecessors = new HashMap<>();
        PriorityQueue<String> queue = new PriorityQueue<>(Comparator.comparingInt(distances::get));

        for (String node : graph.keySet()) {
            distances.put(node, Integer.MAX_VALUE);
            predecessors.put(node, null);
        }

        distances.put(word1, 0);
        queue.add(word1);

        while (!queue.isEmpty()) {
            String current = queue.poll();
            if (current.equals(word2)) break;

            for (Map.Entry<String, Integer> neighbor : graph.get(current).entrySet()) {
                int newDist = distances.get(current) + neighbor.getValue();
                if (newDist < distances.get(neighbor.getKey())) {
                    distances.put(neighbor.getKey(), newDist);
                    predecessors.put(neighbor.getKey(), current);
                    queue.add(neighbor.getKey());
                }
            }
        }

        if (distances.get(word2) == Integer.MAX_VALUE) {
            return "No path from " + word1 + " to " + word2 + "!";
        }

        List<String> path = new LinkedList<>();
        for (String at = word2; at != null; at = predecessors.get(at)) {
            path.add(at);
        }
        Collections.reverse(path);

        return "Shortest path from " + word1 + " to " + word2 + " is: " + String.join(" -> ", path) +
                " (total weight: " + distances.get(word2) + ")";
    }

    private static String randomWalk() {
        List<String> nodes = new ArrayList<>(graph.keySet());
        String current = nodes.get(new Random().nextInt(nodes.size()));
        StringBuilder walk = new StringBuilder(current);

        Set<String> visitedEdges = new HashSet<>();
        while (true) {
            List<String> neighbors = new ArrayList<>(graph.get(current).keySet());
            if (neighbors.isEmpty()) break;

            String next = neighbors.get(new Random().nextInt(neighbors.size()));
            String edge = current + " -> " + next;
            if (visitedEdges.contains(edge)) break;

            visitedEdges.add(edge);
            walk.append(" -> ").append(next);
            current = next;
        }

        try {
            BufferedWriter writer = new BufferedWriter(new FileWriter("random_walk.txt"));
            writer.write(walk.toString());
            writer.close();
        } catch (IOException e) {
            return "Error writing random walk to file: " + e.getMessage();
        }

        return "Random walk: " + walk.toString();
    }
}

```



#### Error stacktrace:

```
scala.collection.Iterator$$anon$19.next(Iterator.scala:973)
	scala.collection.Iterator$$anon$19.next(Iterator.scala:971)
	scala.collection.mutable.MutationTracker$CheckedIterator.next(MutationTracker.scala:76)
	scala.collection.IterableOps.head(Iterable.scala:222)
	scala.collection.IterableOps.head$(Iterable.scala:222)
	scala.collection.AbstractIterable.head(Iterable.scala:933)
	dotty.tools.dotc.interactive.InteractiveDriver.run(InteractiveDriver.scala:168)
	scala.meta.internal.pc.MetalsDriver.run(MetalsDriver.scala:45)
	scala.meta.internal.pc.HoverProvider$.hover(HoverProvider.scala:36)
	scala.meta.internal.pc.ScalaPresentationCompiler.hover$$anonfun$1(ScalaPresentationCompiler.scala:366)
```
#### Short summary: 

java.util.NoSuchElementException: next on empty iterator