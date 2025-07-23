import java.util.*;
import java.io.*;
import java.time.*;

public class TimeTable_scheduler {
    private static final String[] DAYS = {"Mon", "Tues", "Wed", "Thurs", "Fri"};
    private static final String[] TIME_SLOTS = {"8:30-9:25", "9:25-10:20", "10:30-11:25", "11:25-12:20", "Lunch", "1:25-2:20", "2:30-3:25", "3:25-4:20"};
    private static final String[] PROGRAMS = {"BCA-A", "BCA-B", "MCA-A", "MCA-B"};

    private static final String[] FACULTY_INITIALS = {"P", "H", "AP", "GV", "S", "I", "AK", "RM", "SK", "DJ", "NP", "AB", "MK", "TS", "VR"};
    private static final String[] FACULTY_NAMES = {"Prof. Programming", "Prof. HOD", "Prof. Algorithms", "Prof. Data Viz", "Prof. Systems", "Prof. Internet", "Prof. Java", "Prof. Ramesh", "Prof. Sunita", "Prof. Deepak", "Prof. Neha", "Prof. Anil", "Prof. Manish", "Prof. Tanvi", "Prof. Vikram"};

    private static final String[] SUBJECTS = {"AS", "COM", "PSTA", "MLTP", "GD", "DD", "JAVA", "ASAA", "LIB", "MEINTORING"};

    private static Map<String, Map<String, String[]>> timetables = new HashMap<>();
    private static Map<String, Integer> facultyAbsences = new HashMap<>();
    private static Map<String, Integer> facultyLoad = new HashMap<>();
    private static Map<String, List<String>> facultySpecializations = new HashMap<>();

    public static void main(String[] args) {
        initializeData();
        loadOrGenerateTimetable();

        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("\nDepartment Timetable Scheduler");
            System.out.println("1. View Timetable");
            System.out.println("2. Mark Faculty Absence");
            System.out.println("3. Reschedule (Auto-adjust)");
            System.out.println("4. Add Special Booking");
            System.out.println("5. Exit");
            System.out.print("Select option: ");

            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1: displayTimetable(); break;
                case 2: markAbsence(scanner); break;
                case 3: autoAdjustTimetable(); break;
                case 4: addSpecialBooking(scanner); break;
                case 5:
                    saveTimetable();
                    System.out.println("Timetable saved. Exiting...");
                    return;
                default: System.out.println("Invalid choice!");
            }
        }
    }

    private static void initializeData() {
        for (int i = 0; i < FACULTY_INITIALS.length; i++) {
            facultyAbsences.put(FACULTY_INITIALS[i], 0);
            facultyLoad.put(FACULTY_INITIALS[i], 0);
            List<String> specialties = new ArrayList<>(Arrays.asList(SUBJECTS));
            Collections.shuffle(specialties);
            facultySpecializations.put(FACULTY_INITIALS[i], specialties.subList(0, 2 + new Random().nextInt(2)));
        }
        facultySpecializations.put("H", Arrays.asList("COM", "ASAA"));
    }

    private static void loadOrGenerateTimetable() {
        if (!loadTimetableFromFile()) {
            generateNewTimetable();
            System.out.println("Generated new timetable based on department structure");
        }
    }

    private static void generateNewTimetable() {
        timetables = new HashMap<>();
        Random random = new Random();

        for (String program : PROGRAMS) {
            Map<String, String[]> programTimetable = new HashMap<>();
            for (String day : DAYS) {
                String[] dailySlots = new String[TIME_SLOTS.length];
                dailySlots[4] = "Lunch";
                if (day.equals("Mon")) dailySlots[7] = "LIB";
                if (day.equals("Wed")) dailySlots[7] = "LIB";
                if (day.equals("Fri")) dailySlots[7] = "LIB";
                if (day.equals("Mon") || day.equals("Thurs")) {
                    dailySlots[6] = "MEINTORING";
                }
                for (int i = 0; i < dailySlots.length; i++) {
                    if (dailySlots[i] != null) continue;
                    String subject = SUBJECTS[random.nextInt(SUBJECTS.length)];
                    String faculty = findAvailableFaculty(subject, day, i, program, random);
                    dailySlots[i] = (faculty != null) ? subject + "(" + faculty + ")" : subject + "(STAFF)";
                    if (faculty != null) facultyLoad.put(faculty, facultyLoad.get(faculty) + 1);
                }
                programTimetable.put(day, dailySlots);
            }
            timetables.put(program, programTimetable);
        }
    }

    private static String findAvailableFaculty(String subject, String day, int timeSlot, String program, Random random) {
        List<String> suitableFaculty = new ArrayList<>();
        for (String faculty : FACULTY_INITIALS) {
            if (facultySpecializations.get(faculty).contains(subject)) {
                suitableFaculty.add(faculty);
            }
        }
        suitableFaculty.removeIf(f -> !isFacultyAvailable(f, day, timeSlot, program) || facultyAbsences.get(f) >= 2);
        if (suitableFaculty.isEmpty()) {
            for (String faculty : FACULTY_INITIALS) {
                if (isFacultyAvailable(faculty, day, timeSlot, program) && facultyAbsences.get(faculty) < 2) {
                    suitableFaculty.add(faculty);
                }
            }
        }
        if (suitableFaculty.isEmpty()) return null;
        suitableFaculty.sort(Comparator.comparingInt(facultyLoad::get));
        return suitableFaculty.get(random.nextInt(Math.min(3, suitableFaculty.size())));
    }

    private static boolean isFacultyAvailable(String faculty, String day, int timeSlot, String currentProgram) {
        for (String program : PROGRAMS) {
            Map<String, String[]> prog = timetables.get(program);
            if (prog != null) {
                String[] dailySlots = prog.get(day);
                if (dailySlots != null && dailySlots[timeSlot] != null && dailySlots[timeSlot].contains("(" + faculty + ")")) {
                    return false;
                }
            }
        }
        if (faculty.equals("H")) {
            int hodClasses = 0;
            for (String program : PROGRAMS) {
                Map<String, String[]> prog = timetables.get(program);
                if (prog != null) {
                    String[] dailySlots = prog.get(day);
                    if (dailySlots != null) {
                        for (String slot : dailySlots) {
                            if (slot != null && slot.contains("(H)")) hodClasses++;
                        }
                    }
                }
            }
            if (hodClasses >= 2) return false;
        }
        return true;
    }

    private static boolean loadTimetableFromFile() {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("timetable.dat"))) {
            timetables = (Map<String, Map<String, String[]>>) ois.readObject();
            facultyAbsences = (Map<String, Integer>) ois.readObject();
            facultyLoad = (Map<String, Integer>) ois.readObject();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    private static void saveTimetable() {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("timetable.dat"))) {
            oos.writeObject(timetables);
            oos.writeObject(facultyAbsences);
            oos.writeObject(facultyLoad);
        } catch (IOException e) {
            System.out.println("Error saving timetable: " + e.getMessage());
        }
    }
}
