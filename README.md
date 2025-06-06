# Market-Entry-Feasibility-Analyzer

import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.List;
import java.util.ArrayList;

public class MarketEntryAnalyzer extends JFrame {

    // Data Models 
    static class Country {
        String code, name;
        int easeOfBusiness, politicalStability, risk;

        Country(String code, String name, int ease, int stability, int risk) {
            this.code = code;
            this.name = name;
            this.easeOfBusiness = ease;
            this.politicalStability = stability;
            this.risk = risk;
        }

        public String toString() { return name; }
    }

    static class Industry {
        String name;
        double marketGrowthRate;

        Industry(String name, double growthRate) {
            this.name = name;
            this.marketGrowthRate = growthRate;
        }

        public String toString() { return name; }
    }

    static class FeasibilityResult {
        double score;
        String riskLevel;
        String recommendation;

        FeasibilityResult(double score, String risk, String recommendation) {
            this.score = score;
            this.riskLevel = risk;
            this.recommendation = recommendation;
        }

        public String toString() {
            return String.format("Score: %.2f\nRisk Level: %s\nRecommendation: %s",
                score, riskLevel, recommendation);
        }
    }

    // File Loaders 
    private List<Country> loadCountries(String filename) {
        List<Country> countries = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(",");
                countries.add(new Country(parts[0], parts[1],
                    Integer.parseInt(parts[2]),
                    Integer.parseInt(parts[3]),
                    Integer.parseInt(parts[4])));
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error loading countries: " + e.getMessage());
        }
        return countries;
    }

    private List<Industry> loadIndustries(String filename) {
        List<Industry> industries = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(",");
                industries.add(new Industry(parts[0], Double.parseDouble(parts[1])));
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error loading industries: " + e.getMessage());
        }
        return industries;
    }

    //  Analysis Logic 
    private FeasibilityResult analyze(Country c, Industry i, int strategyStrength) {
        double baseScore = (c.easeOfBusiness * 0.4) + (c.politicalStability * 0.3) +
                           (i.marketGrowthRate * 2) + (strategyStrength * 0.3) - (c.risk * 0.5);

        String riskLevel = c.risk >= 50 ? "High" : c.risk >= 25 ? "Medium" : "Low";
        String recommendation = baseScore >= 75 ? "Strongly Recommended"
                             : baseScore >= 50 ? "Consider with Caution"
                             : "Not Recommended";

        return new FeasibilityResult(baseScore, riskLevel, recommendation);
    }

    //  GUI Setup (use Jgrasp because Eclipse didnt work with me)
    public MarketEntryAnalyzer() {
        setTitle("🌍 Market Entry Feasibility Analyzer");
        setSize(750, 550);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Try to set system look and feel
        try {
            UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            System.out.println("Default look and feel applied.");
        }

        // Load data
        List<Country> countries = loadCountries("countries.txt");
        List<Industry> industries = loadIndustries("industries.txt");

        JComboBox<Country> countryBox = new JComboBox<>(countries.toArray(new Country[0]));
        JComboBox<Industry> industryBox = new JComboBox<>(industries.toArray(new Industry[0]));

        JSlider strategySlider = new JSlider(0, 100, 50);
        strategySlider.setMajorTickSpacing(25);
        strategySlider.setPaintTicks(true);
        strategySlider.setPaintLabels(true);

        JButton analyzeBtn = new JButton("📈 Analyze Entry");
        JTextArea resultArea = new JTextArea(6, 40);
        resultArea.setEditable(false);
        resultArea.setLineWrap(true);
        resultArea.setWrapStyleWord(true);
        resultArea.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        JScrollPane scrollPane = new JScrollPane(resultArea);

        // Layout and Styling
        JPanel inputPanel = new JPanel();
        inputPanel.setLayout(new BoxLayout(inputPanel, BoxLayout.Y_AXIS));
        inputPanel.setBorder(new EmptyBorder(20, 20, 20, 20));

        inputPanel.add(labeledComponent("🌐 Country:", countryBox));
        inputPanel.add(Box.createVerticalStrut(10));
        inputPanel.add(labeledComponent("🏭 Industry:", industryBox));
        inputPanel.add(Box.createVerticalStrut(10));
        inputPanel.add(labeledComponent("🚀 Strategy Strength:", strategySlider));
        inputPanel.add(Box.createVerticalStrut(20));
        inputPanel.add(analyzeBtn);

        JLabel title = new JLabel("📊 Market Feasibility Result", SwingConstants.CENTER);
        title.setFont(new Font("Segoe UI", Font.BOLD, 18));
        title.setBorder(new EmptyBorder(10, 0, 10, 0));

        JPanel centerPanel = new JPanel(new BorderLayout());
        centerPanel.setBorder(new EmptyBorder(10, 20, 20, 20));
        centerPanel.add(title, BorderLayout.NORTH);
        centerPanel.add(scrollPane, BorderLayout.CENTER);

        add(inputPanel, BorderLayout.NORTH);
        add(centerPanel, BorderLayout.CENTER);

        analyzeBtn.addActionListener(e -> {
            Country c = (Country) countryBox.getSelectedItem();
            Industry i = (Industry) industryBox.getSelectedItem();
            int s = strategySlider.getValue();
            FeasibilityResult result = analyze(c, i, s);
            resultArea.setText(result.toString());
        });
    }

    private JPanel labeledComponent(String labelText, JComponent component) {
        JPanel panel = new JPanel(new BorderLayout());
        JLabel label = new JLabel(labelText);
        label.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        panel.add(label, BorderLayout.NORTH);
        panel.add(component, BorderLayout.CENTER);
        return panel;
    }

    // === Main ===
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MarketEntryAnalyzer().setVisible(true));
    }
}
