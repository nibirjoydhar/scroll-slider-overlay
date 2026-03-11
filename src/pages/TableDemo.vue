<script setup lang="ts">
import OverlayScrollbar from '../components/ui/OverlayScrollbar.vue'

/**
 * TableDemo Component
 * 
 * Demonstrates the OverlayScrollbar component with a large data table
 * that triggers vertical scrolling.
 */

// Generate sample data with 150 rows
interface TableRow {
  id: number
  name: string
  email: string
  status: string
  joinDate: string
  amount: number
}

const generateTableData = (): TableRow[] => {
  type StatusType = 'Active' | 'Inactive' | 'Pending'
  const statuses: StatusType[] = ['Active', 'Inactive', 'Pending']
  
  const rows: TableRow[] = []
  for (let i = 0; i < 100; i++) {
    const statusIndex = Math.floor(Math.random() * 3)
    const status: StatusType = statuses[statusIndex] as StatusType
    rows.push({
      id: i + 1,
      name: `User ${i + 1}`,
      email: `user${i + 1}@example.com`,
      status: status,
      joinDate: new Date(2024, Math.floor(Math.random() * 12), Math.floor(Math.random() * 28) + 1).toLocaleDateString(),
      amount: Math.floor(Math.random() * 10000) + 100,
    })
  }
  return rows
}

const tableData = generateTableData()
</script>

<template>
  <div class="page-wrapper">
    <!-- Header section -->
    <div class="page-header">
      <h1>Data Table with Custom Scrollbar</h1>
      <p class="subtitle">Scroll the table below to see the custom overlay scrollbar in action</p>
    </div>

    <!-- Main data table container -->
    <div class="data-table-container">
      <!-- OverlayScrollbar wrapping the table -->
      <OverlayScrollbar class="scrollable-table">
        <table class="data-table">
          <thead class="table-head">
            <tr>
              <th>ID</th>
              <th>Name</th>
              <th>Email</th>
              <th>Status</th>
              <th>Join Date</th>
              <th>Amount</th>
            </tr>
          </thead>
          <tbody class="table-body">
            <tr v-for="row in tableData" :key="row.id" class="table-row">
              <td class="cell">{{ row.id }}</td>
              <td class="cell">{{ row.name }}</td>
              <td class="cell">{{ row.email }}</td>
              <td class="cell">
                <span class="status" :class="`status-${row.status.toLowerCase()}`">
                  {{ row.status }}
                </span>
              </td>
              <td class="cell">{{ row.joinDate }}</td>
              <td class="cell amount">${{ row.amount.toLocaleString() }}</td>
            </tr>
          </tbody>
        </table>
      </OverlayScrollbar>
    </div>

    <!-- Footer info -->
    <div class="page-footer">
      <h3>Features</h3>
      <ul>
        <li>✓ Fixed track, dynamic thumb height</li>
        <li>✓ Thumb moves with scroll position</li>
        <li>✓ Draggable scrollbar thumb</li>
        <li>✓ Smooth animations</li>
        <li>✓ No layout shift</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
/* Page wrapper */
.page-wrapper {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
}

/* Header */
.page-header {
  padding: 2rem;
  background: rgba(255, 255, 255, 0.95);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.page-header h1 {
  margin: 0 0 0.5rem;
  font-size: 2rem;
  color: #1a202c;
  font-weight: 700;
}

.subtitle {
  margin: 0;
  font-size: 1rem;
  color: #4a5568;
}

/* Data table container - main scrollable area */
.data-table-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 2rem;
  overflow: hidden;
}

/* Scrollable table inside the container */
.scrollable-table {
  flex: 1;
  background: white;
  border-radius: 8px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1),
    0 10px 10px -5px rgba(0, 0, 0, 0.04);
  overflow: hidden;
}

/* Table styles */
.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
}

.table-head {
  background-color: #f8fafc;
  position: sticky;
  top: 0;
  z-index: 10;
}

.table-head th {
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  color: #1a202c;
  border-bottom: 2px solid #e2e8f0;
  white-space: nowrap;
}

.table-body {
  background: white;
}

.table-row {
  border-bottom: 1px solid #e2e8f0;
  transition: background-color 0.15s ease;
}

.table-row:hover {
  background-color: #f8fafc;
}

.table-row td {
  padding: 1rem;
}

.cell {
  color: #2d3748;
  vertical-align: middle;
}

.cell.amount {
  text-align: right;
  font-weight: 500;
  color: #2b6cb0;
}

/* Status badge */
.status {
  display: inline-block;
  padding: 0.35rem 0.75rem;
  border-radius: 4px;
  font-size: 0.85rem;
  font-weight: 500;
  width: fit-content;
}

.status-active {
  background-color: #dcfce7;
  color: #166534;
}

.status-inactive {
  background-color: #fee2e2;
  color: #991b1b;
}

.status-pending {
  background-color: #fef3c7;
  color: #92400e;
}

/* Footer info */
.page-footer {
  padding: 2rem;
  background: rgba(255, 255, 255, 0.95);
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.page-footer h3 {
  margin: 0 0 1rem;
  color: #1a202c;
  font-size: 1.1rem;
  font-weight: 600;
}

.page-footer ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.page-footer li {
  color: #4a5568;
  font-size: 0.95rem;
}

/* Responsive design */
@media (max-width: 768px) {
  .page-wrapper {
    height: auto;
  }

  .page-header {
    padding: 1rem;
  }

  .page-header h1 {
    font-size: 1.5rem;
  }

  .subtitle {
    font-size: 0.9rem;
  }

  .data-table-container {
    padding: 1rem;
    height: 600px;
  }

  .table-head th {
    padding: 0.75rem 0.5rem;
    font-size: 0.9rem;
  }

  .table-row td {
    padding: 0.75rem 0.5rem;
    font-size: 0.9rem;
  }

  .page-footer {
    padding: 1rem;
  }

  .page-footer ul {
    grid-template-columns: 1fr;
  }
}
</style>
