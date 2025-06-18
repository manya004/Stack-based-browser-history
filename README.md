# Stack-based-browser-history
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.net.URI;
import java.util.*;
import java.util.List;

public class PerfectBrowserHistory {

    private Stack<String> backStack = new Stack<>();
    private Stack<String> forwardStack = new Stack<>();
    private Map<String, Integer> visitCount = new HashMap<>();
    private Set<String> bookmarks = new LinkedHashSet<>();

    private String currentUrl = "";
    private boolean darkMode = false;

    private JFrame frame;
    private JTextField urlField;
    private JTextArea activityArea;
    private JLabel currentPageLabel;
    private JPanel mainPanel;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(PerfectBrowserHistory::new);
    }

    public PerfectBrowserHistory() {
        frame = new JFrame("🌐 Perfect Browser History System");
        frame.setSize(1000, 600);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);

        mainPanel = new JPanel(new BorderLayout(10, 10));
        mainPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        JLabel title = new JLabel("🌐 Stack-Based Browser History System", JLabel.CENTER);
        title.setFont(new Font("Segoe UI", Font.BOLD, 24));
        mainPanel.add(title, BorderLayout.NORTH);

        // Left Activity Panel
        activityArea = new JTextArea();
        activityArea.setEditable(false);
        activityArea.setFont(new Font("Consolas", Font.PLAIN, 14));
        JScrollPane scrollPane = new JScrollPane(activityArea);
        scrollPane.setBorder(BorderFactory.createTitledBorder("Activity Log"));
        scrollPane.setPreferredSize(new Dimension(350, 500));
        mainPanel.add(scrollPane, BorderLayout.WEST);

        // Right Panel
        JPanel rightPanel = new JPanel(new BorderLayout(10, 10));
        urlField = new JTextField();
        urlField.setFont(new Font("Segoe UI", Font.PLAIN, 16));
        urlField.setBorder(BorderFactory.createTitledBorder("Enter URL"));
        rightPanel.add(urlField, BorderLayout.NORTH);

        JPanel buttonPanel = new JPanel(new GridLayout(5, 2, 10, 10));
        String[] buttons = {
            "Visit", "Back", "Forward", "Bookmark", "Delete Bookmark",
            "Most Visited", "Search URL", "Clear History", "Toggle Dark Mode", "Exit"
        };

        for (String label : buttons) {
            JButton button = createStyledButton(label);
            buttonPanel.add(button);
            assignAction(button, label);
        }

        rightPanel.add(buttonPanel, BorderLayout.CENTER);

        currentPageLabel = new JLabel("Current Page: None");
        currentPageLabel.setFont(new Font("Segoe UI", Font.ITALIC, 14));
        rightPanel.add(currentPageLabel, BorderLayout.SOUTH);

        mainPanel.add(rightPanel, BorderLayout.CENTER);
        frame.add(mainPanel);
        frame.setVisible(true);
    }

    private JButton createStyledButton(String text) {
        JButton button = new JButton(text);
        button.setFocusPainted(false);
        button.setBackground(new Color(65, 105, 225));
        button.setForeground(Color.WHITE);
        button.setFont(new Font("Segoe UI", Font.BOLD, 13));
        button.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        button.addMouseListener(new MouseAdapter() {
            public void mouseEntered(MouseEvent e) {
                button.setBackground(new Color(30, 144, 255));
            }

            public void mouseExited(MouseEvent e) {
                button.setBackground(new Color(65, 105, 225));
            }
        });
        return button;
    }

    private void assignAction(JButton button, String label) {
        switch (label) {
            case "Visit" -> button.addActionListener(e -> visitUrl());
            case "Back" -> button.addActionListener(e -> goBack());
            case "Forward" -> button.addActionListener(e -> goForward());
            case "Bookmark" -> button.addActionListener(e -> bookmark());
            case "Delete Bookmark" -> button.addActionListener(e -> deleteBookmark());
            case "Most Visited" -> button.addActionListener(e -> showMostVisited());
            case "Search URL" -> button.addActionListener(e -> searchUrl());
            case "Clear History" -> button.addActionListener(e -> clearHistory());
            case "Toggle Dark Mode" -> button.addActionListener(e -> toggleDarkMode());
            case "Exit" -> button.addActionListener(e -> System.exit(0));
        }
    }

    private void visitUrl() {
    String url = urlField.getText().trim();
    if (url.isEmpty()) return;

    if (!url.startsWith("http")) url = "http://" + url;

    if (!currentUrl.isEmpty()) backStack.push(currentUrl);
    currentUrl = url;
    forwardStack.clear();
    visitCount.put(url, visitCount.getOrDefault(url, 0) + 1);

    openUrl(url);
    urlField.setText(currentUrl); // ✅ update field with current URL
    log("Visited: " + url);
    updateCurrentPageLabel();
}


    private void openUrl(String url) {
        try {
            Desktop.getDesktop().browse(new URI(url));
        } catch (Exception e) {
            log("⚠️ Failed to open URL.");
        }
    }

    private void goBack() {
    if (!backStack.isEmpty()) {
        forwardStack.push(currentUrl);
        currentUrl = backStack.pop();
        openUrl(currentUrl);
        urlField.setText(currentUrl); // ✅ reflect in field
        log("⬅️ Back to: " + currentUrl);
        updateCurrentPageLabel();
    } else {
        log("No Back History.");
    }
}


    private void goForward() {
    if (!forwardStack.isEmpty()) {
        backStack.push(currentUrl);
        currentUrl = forwardStack.pop();
        openUrl(currentUrl);
        urlField.setText(currentUrl); // ✅ reflect in field
        log("➡️ Forward to: " + currentUrl);
        updateCurrentPageLabel();
    } else {
        log("No Forward History.");
    }
}


    private void bookmark() {
        if (!currentUrl.isEmpty()) {
            bookmarks.add(currentUrl);
            log("🔖 Bookmarked: " + currentUrl);
        }
    }

    private void deleteBookmark() {
        if (bookmarks.isEmpty()) {
            log("No bookmarks to delete.");
            return;
        }
        String[] bmArray = bookmarks.toArray(new String[0]);
        String selected = (String) JOptionPane.showInputDialog(frame, "Select bookmark to delete:", "Delete Bookmark", JOptionPane.PLAIN_MESSAGE, null, bmArray, bmArray[0]);
        if (selected != null) {
            bookmarks.remove(selected);
            log("❌ Bookmark deleted: " + selected);
        }
    }

    private void showMostVisited() {
        if (visitCount.isEmpty()) {
            log("No pages visited yet.");
            return;
        }
        String mostVisited = Collections.max(visitCount.entrySet(), Map.Entry.comparingByValue()).getKey();
        int count = visitCount.get(mostVisited);
        log("🔥 Most Visited: " + mostVisited + " (" + count + " times)");
    }

    private void searchUrl() {
        String keyword = JOptionPane.showInputDialog(frame, "Enter keyword to search:");
        if (keyword == null || keyword.isEmpty()) return;

        keyword = keyword.toLowerCase();
        Set<String> allUrls = new LinkedHashSet<>();
        allUrls.addAll(backStack);
        allUrls.addAll(forwardStack);
        allUrls.addAll(bookmarks);
        if (!currentUrl.isEmpty()) allUrls.add(currentUrl);

        List<String> found = new ArrayList<>();
        for (String url : allUrls) {
            if (kmpSearch(url.toLowerCase(), keyword)) {
                found.add(url);
            }
        }

        if (found.isEmpty()) {
            log("No match found for keyword: " + keyword);
        } else {
            log("🔍 Results for '" + keyword + "':");
            for (String res : found) {
                log("   → " + res);
            }
        }
    }

    private int[] buildLPS(String pattern) {
        int[] lps = new int[pattern.length()];
        int len = 0;
        int i = 1;
        while (i < pattern.length()) {
            if (pattern.charAt(i) == pattern.charAt(len)) {
                lps[i++] = ++len;
            } else if (len != 0) {
                len = lps[len - 1];
            } else {
                lps[i++] = 0;
            }
        }
        return lps;
    }

    private boolean kmpSearch(String text, String pattern) {
        int[] lps = buildLPS(pattern);
        int i = 0, j = 0;
        while (i < text.length()) {
            if (text.charAt(i) == pattern.charAt(j)) {
                i++;
                j++;
                if (j == pattern.length()) return true;
            } else if (j > 0) {
                j = lps[j - 1];
            } else {
                i++;
            }
        }
        return false;
    }

    private void clearHistory() {
    backStack.clear();
    forwardStack.clear();
    currentUrl = "";
    visitCount.clear();
    urlField.setText(""); // ✅ clear field
    updateCurrentPageLabel();
    log("🧹 History cleared.");
}


    private void toggleDarkMode() {
        darkMode = !darkMode;
        Color bg = darkMode ? new Color(40, 40, 40) : Color.WHITE;
        Color fg = darkMode ? Color.WHITE : Color.BLACK;

        mainPanel.setBackground(bg);
        urlField.setBackground(bg);
        urlField.setForeground(fg);
        activityArea.setBackground(bg);
        activityArea.setForeground(fg);
        currentPageLabel.setForeground(fg);
        log("🌗 Dark Mode " + (darkMode ? "Enabled" : "Disabled"));
    }

    private void updateCurrentPageLabel() {
        currentPageLabel.setText("Current Page: " + (currentUrl.isEmpty() ? "None" : currentUrl));
    }

    private void log(String message) {
        activityArea.append(message + "\n");
    }
}
