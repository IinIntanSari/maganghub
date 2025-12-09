<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Employee Management</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        .hidden { display: none !important; }
        /* Custom modal for form */
        #form-modal {
            position: fixed; left: 0; top: 0; width: 100%; height: 100%;
            display: flex; align-items: center; justify-content: center;
            background: rgba(0,0,0,0.4);
            z-index: 1050;
        }
        #employee-form {
            background: #fff; padding: 20px; border-radius: 8px; width: 100%; max-width: 400px;
        }
    </style>
</head>
<body>
    <header class="bg-primary text-white p-3 mb-4">
        <h1 class="h3">Employee Management</h1>
    </header>

    <main class="container">
        <section id="controls" class="d-flex flex-wrap gap-2 mb-3">
            <input id="search" class="form-control me-2" style="max-width: 300px;" placeholder="Search by name, position, department" />
            <select id="filter-dept" class="form-select" style="max-width: 200px;"><option value="">All departments</option></select>
            <select id="filter-status" class="form-select" style="max-width: 200px;">
                <option value="">All status</option>
                <option value="active">Active</option>
                <option value="inactive">Inactive</option>
            </select>
            <button id="add-new" class="btn btn-success ms-auto">Add New Employee</button>
        </section>

        <section id="list">
            <div class="table-responsive">
                <table id="employees-table" class="table table-striped table-hover bg-white">
                    <thead class="table-dark">
                        <tr>
                            <th>ID</th><th>Name</th><th>Email</th><th>Position</th>
                            <th>Department</th><th>Salary</th><th>Hire Date</th><th>Status</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        </tbody>
                </table>
            </div>
        </section>

        <section id="form-modal" class="hidden">
            <form id="employee-form" class="shadow-lg">
                <h2 id="form-title" class="mb-3">Add Employee</h2>
                <input name="id" type="hidden" />

                <div class="mb-3"><label class="form-label">Name<input name="name" class="form-control" required /></label></div>
                <div class="mb-3"><label class="form-label">Email<input name="email" type="email" class="form-control" required /></label></div>
                <div class="mb-3"><label class="form-label">Position<input name="position" class="form-control" required /></label></div>
                <div class="mb-3"><label class="form-label">Department<input name="department" class="form-control" required /></label></div>
                <div class="mb-3"><label class="form-label">Salary<input name="salary" type="number" min="1" class="form-control" required /></label></div>
                <div class="mb-3"><label class="form-label">Hire Date<input name="hire_date" type="date" class="form-control" required /></label></div>
                <div class="mb-3">
                    <label class="form-label">Status
                        <select name="status" class="form-select">
                            <option value="active">active</option>
                            <option value="inactive">inactive</option>
                        </select>
                    </label>
                </div>
                
                <div class="form-actions d-flex justify-content-end gap-2 mt-4">
                    <button type="button" id="cancel" class="btn btn-secondary">Cancel</button>
                    <button type="submit" class="btn btn-primary">Save</button>
                </div>
            </form>
        </section>
    </main>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="app.js"></script>
</body>
</html> maganghub
