import SwiftUI

// ===============================
// Data Models (Equivalent to React Interfaces)
// ===============================

// Enum for Production Status
enum ProductionStatus: String, CaseIterable, Identifiable, Codable {
    case planning = "Planning"
    case inProduction = "In Production"
    case finished = "Finished"
    case sandblasted = "Sandblasted"

    var id: String { self.rawValue }

    // Helper for color coding
    var color: Color {
        switch self {
        case .planning: return .gray
        case .inProduction: return .yellow
        case .finished: return .blue
        case .sandblasted: return .teal
        }
    }
    // Helper for icon
    var icon: String {
        return "hammer" // SF Symbol
    }
}

// Enum for Project Application Status
enum ProjectAppStatus: String, CaseIterable, Identifiable, Codable {
    case active = "Active"
    case onHold = "On Hold"
    case completed = "Completed"
    case cancelled = "Cancelled"

    var id: String { self.rawValue }

    // Helper for color coding
    var color: Color {
        switch self {
        case .active: return .green
        case .onHold: return .orange
        case .completed: return .blue
        case .cancelled: return .red
        }
    }
     // Helper for icon
    var icon: String {
        return "circle.dotted" // SF Symbol
    }
}

// Project Struct (Conforms to Identifiable for Lists, Codable for potential saving)
struct Project: Identifiable, Codable {
    var id: String = UUID().uuidString // Use UUID for default ID
    var projectNumber: String
    var projectName: String
    var productionStatus: ProductionStatus
    var inventoryReady: Int // Units ready for loading
    var completionRate: Double // Use Double for ProgressView (0.0 - 1.0)
    var dailyDeliveries: Int // Units delivered today
    var dailyInstallationReport: String // Summary or link
    var projectStatus: ProjectAppStatus
    var equipmentCount: Int

    // Helper to get completion rate as percentage string
    var completionPercentageString: String {
        String(format: "%.0f%%", completionRate * 100)
    }
}

// ===============================
// View Model (Handles Data and Logic)
// ===============================

class ProjectViewModel: ObservableObject {
    @Published var projects: [Project] = []
    @Published var showingCompletionAlert = false
    @Published var completedProjectName: String? = nil

    // Store initial state for comparison (similar purpose to React version)
    private var initialProjectStates: [String: Project] = [:]

    init() {
        loadMockData()
        // Store initial states after loading
        projects.forEach { initialProjectStates[$0.id] = $0 }
    }

    // Load Mock Data (Equivalent to React initialProjects)
    func loadMockData() {
        projects = [
            Project(projectNumber: "PC-001", projectName: "Downtown Tower", productionStatus: .finished, inventoryReady: 150, completionRate: 0.75, dailyDeliveries: 20, dailyInstallationReport: "Section A complete.", projectStatus: .active, equipmentCount: 5),
            Project(projectNumber: "PC-002", projectName: "West Bridge", productionStatus: .sandblasted, inventoryReady: 80, completionRate: 1.0, dailyDeliveries: 0, dailyInstallationReport: "All units installed.", projectStatus: .completed, equipmentCount: 0),
            Project(projectNumber: "PC-003", projectName: "North Mall", productionStatus: .inProduction, inventoryReady: 30, completionRate: 0.20, dailyDeliveries: 5, dailyInstallationReport: "Foundation prep.", projectStatus: .active, equipmentCount: 8),
            Project(projectNumber: "PC-004", projectName: "Airport Hangar", productionStatus: .planning, inventoryReady: 0, completionRate: 0.0, dailyDeliveries: 0, dailyInstallationReport: "Awaiting permits.", projectStatus: .onHold, equipmentCount: 2)
        ]
    }

    // Add a new project
    func addProject() {
        let newProject = Project(
            projectNumber: "PC-\(String(format: "%03d", projects.count + 1))",
            projectName: "New Project",
            productionStatus: .planning,
            inventoryReady: 0,
            completionRate: 0.0,
            dailyDeliveries: 0,
            dailyInstallationReport: "Not Started",
            projectStatus: .active,
            equipmentCount: 0
        )
        projects.append(newProject)
        // Add to initial states as well if needed for completion check logic
        initialProjectStates[newProject.id] = newProject
        // TODO: Add notification/feedback to user
    }

    // Delete projects at specified offsets
    func deleteProject(at offsets: IndexSet) {
        let idsToDelete = offsets.map { projects[$0].id }
        projects.remove(atOffsets: offsets)
        // Remove from initial states if they exist
        idsToDelete.forEach { initialProjectStates.removeValue(forKey: $0) }
        // TODO: Add notification/feedback to user
    }

    // Update a project (simplified example - directly modify)
    // In a real app, you might have a dedicated editing view/modal
    func updateProject(id: String, updates: (inout Project) -> Void) {
        if let index = projects.firstIndex(where: { $0.id == id }) {
            let originalCompletion = projects[index].completionRate
            updates(&projects[index]) // Apply the updates via closure

            // Check for completion *after* update
            let updatedProject = projects[index]
            if updatedProject.completionRate == 1.0 && originalCompletion < 1.0 {
                // Mark as completed if not already
                if updatedProject.projectStatus != .completed {
                    projects[index].projectStatus = .completed
                }
                // Trigger completion alert/confetti
                triggerCompletionAlert(projectName: updatedProject.projectName)
            }
        }
    }

    // Trigger the completion alert
    func triggerCompletionAlert(projectName: String) {
        completedProjectName = projectName
        showingCompletionAlert = true
    }

    // Simulate requesting materials
    func requestMaterials(project: Project) {
        print("Requesting materials for Project ID: \(project.id), Name: \(project.projectName)")
        // In a real app: Show alert, navigate, or call API
    }

    // Simulate ordering items
    func orderItems() {
        print("Order Items button tapped")
        // In a real app: Show alert, navigate, or call API
    }
}

// ===============================
// SwiftUI Views
// ===============================

// View for a single row in the list
struct ProjectRowView: View {
    let project: Project

    var body: some View {
        HStack(alignment: .top, spacing: 10) {
            VStack(alignment: .leading, spacing: 6) {
                // Project Name & Number
                HStack {
                    Image(systemName: "number")
                        .foregroundColor(.gray)
                    Text(project.projectNumber)
                        .font(.caption)
                        .foregroundColor(.gray)
                }
                HStack {
                     Image(systemName: "building.2")
                        .foregroundColor(.secondary)
                    Text(project.projectName)
                        .font(.headline)
                        .foregroundColor(.primary) // Use primary/secondary for adaptability
                }

                // Production Status
                HStack {
                    Image(systemName: project.productionStatus.icon)
                    Text(project.productionStatus.rawValue)
                }
                .font(.caption)
                .padding(.horizontal, 6)
                .padding(.vertical, 3)
                .background(project.productionStatus.color.opacity(0.2))
                .foregroundColor(project.productionStatus.color)
                .cornerRadius(5)

                 // Project Status
                HStack {
                    Image(systemName: project.projectStatus.icon)
                    Text(project.projectStatus.rawValue)
                }
                .font(.caption)
                .foregroundColor(project.projectStatus.color)


            }

            Spacer() // Pushes content to sides

            VStack(alignment: .trailing, spacing: 6) {
                // Completion Rate
                HStack {
                    ProgressView(value: project.completionRate)
                        .frame(width: 80) // Adjust width as needed
                    Text(project.completionPercentageString)
                        .font(.caption.weight(.medium))
                }

                // Inventory
                HStack {
                    Text("\(project.inventoryReady)")
                    Image(systemName: "shippingbox")
                }
                .font(.caption)
                .foregroundColor(.secondary)

                // Deliveries
                HStack {
                    Text("\(project.dailyDeliveries)")
                    Image(systemName: "truck")
                }
                .font(.caption)
                .foregroundColor(.secondary)

                // Equipment
                HStack {
                    Text("\(project.equipmentCount)")
                    Image(systemName: "wrench")
                }
                .font(.caption)
                .foregroundColor(.secondary)
            }
        }
        .padding(.vertical, 5) // Add some padding to the row
    }
}

// Main List View
struct ProjectListView: View {
    // Use @StateObject for the ViewModel lifecycle
    @StateObject private var viewModel = ProjectViewModel()

    var body: some View {
        NavigationView {
            List {
                // Loop through projects and display rows
                ForEach(viewModel.projects) { project in
                    // Wrap row in NavigationLink if detail view is needed later
                    // NavigationLink(destination: ProjectDetailView(project: project)) {
                        ProjectRowView(project: project)
                    // }
                     // Add context menu for actions like requesting materials
                     .contextMenu {
                         Button {
                             viewModel.requestMaterials(project: project)
                         } label: {
                             Label("Request Materials", systemImage: "doc.text")
                         }
                         // Add other actions if needed
                     }
                }
                .onDelete(perform: viewModel.deleteProject) // Enable swipe-to-delete
            }
            .navigationTitle("ADVANCE PROJECTS") // Set the title
            .listStyle(.plain) // Use plain list style
            .toolbar {
                // Edit button for list reordering/deleting
                ToolbarItem(placement: .navigationBarLeading) {
                    EditButton()
                }
                // Buttons for adding project and ordering items
                ToolbarItemGroup(placement: .navigationBarTrailing) {
                    Button {
                        viewModel.orderItems()
                    } label: {
                        Label("Order Items", systemImage: "cart") // SF Symbol for cart
                    }

                    Button {
                        viewModel.addProject()
                    } label: {
                        Label("Add Project", systemImage: "plus") // SF Symbol for plus
                    }
                }
            }
            // Alert for project completion
            .alert(isPresented: $viewModel.showingCompletionAlert) {
                Alert(
                    title: Text("Project Completed! 🎉"),
                    message: Text("\(viewModel.completedProjectName ?? "A project") has reached 100% completion."),
                    dismissButton: .default(Text("OK")) {
                        viewModel.completedProjectName = nil // Reset name after dismissal
                    }
                )
            }
            // Add overlay for logo if needed (using ZStack or .background)
            // Placeholder for logo - requires adding an Image view
             .overlay(alignment: .topLeading) {
                 // Replace with your actual logo loading logic
                 Image("logo_placeholder") // Assumes you have an image named "logo_placeholder" in your Assets.xcassets
                     .resizable()
                     .scaledToFit()
                     .frame(height: 30) // Adjust size
                     .padding(.leading) // Adjust padding
                     .opacity(0.5) // Example styling
                     .allowsHitTesting(false) // Prevent interaction
             }
        }
        // Apply a background color if desired (adaptable dark/light mode)
        // .background(Color(UIColor.systemGroupedBackground))
    }
}

// ===============================
// App Entry Point
// ===============================

// @main
struct PrecastTrackerApp: App { // Rename if needed
    var body: some Scene {
        WindowGroup {
            ProjectListView()
        }
    }
}

// ===============================
// Previews (for Xcode Canvas)
// ===============================

struct ProjectListView_Previews: PreviewProvider {
    static var previews: some View {
        ProjectListView()
            .preferredColorScheme(.dark) // Preview in dark mode
    }
}

struct ProjectRowView_Previews: PreviewProvider {
    static var previews: some View {
        ProjectRowView(project: ProjectViewModel().projects[0])
            .padding()
            .previewLayout(.sizeThatFits)

        ProjectRowView(project: ProjectViewModel().projects[1])
            .padding()
            .previewLayout(.sizeThatFits)
            .preferredColorScheme(.dark)
    }
}

// Note: To use this code:
// 1. Create a new Xcode project, selecting the "App" template under the iOS tab.
// 2. Choose "SwiftUI" for the Interface and "Swift" for the Language.
// 3. Replace the contents of the main ContentView.swift (or similarly named file) and the App file (e.g., YourAppNameApp.swift) with this code.
// 4. Add a placeholder image named "logo_placeholder" to your Assets.xcassets file or remove/replace the .overlay code block in ProjectListView.
// 5. Build and run on a simulator or device.
