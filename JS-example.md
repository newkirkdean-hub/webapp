# JavaScript Member Search Example

This document contains a JavaScript example for implementing a member search and selection interface.

## Overview

This code provides functionality for:
- Fetching member data from a servlet endpoint
- Displaying members in a searchable table
- Selecting individual members
- Handling user interactions (search, select, cancel)

## Complete Code

```javascript
let allMembers = [];
let filteredMembers = [];
let selectedMember = null;

// Fetch members from servlet
async function loadMembers() {
    try {
        const response = await fetch('../../searchmembers');
        if (!response.ok) {
            throw new Error('Failed to fetch members');
        }
        allMembers = await response.json();
        filteredMembers = [...allMembers];
        renderTable(filteredMembers);
    } catch (error) {
        console.error('Error loading members:', error);
        document.getElementById('tableBody').innerHTML = 
            '<tr><td colspan="4" class="error">Error loading members. Please try again.</td></tr>'; 
    }
}

// Render table with member data
function renderTable(members) {
    const tbody = document.getElementById('tableBody');
    
    if (members.length === 0) {
        tbody.innerHTML = '<tr><td colspan="4" class="no-results">No members found</td></tr>';
        return;
    }

    tbody.innerHTML = members.map((member, index) => `
        <tr data-index="${index}" data-id="${member.ID}">
            <td>${escapeHtml(member.First_Name || '')}</td>
            <td>${escapeHtml(member.Last_Name || '')}</td>
            <td>${escapeHtml(member.Address || '')}</td>
            <td>${escapeHtml(member.Customer_Card_Number || '')}</td>
        </tr>
    `).join('');

    // Add click listeners to rows
    tbody.querySelectorAll('tr').forEach(row => {
        row.addEventListener('click', () => selectRow(row));
    });
}

// Select a table row
function selectRow(row) {
    // Remove previous selection
    document.querySelectorAll('#tableBody tr').forEach(r => r.classList.remove('selected'));
    
    // Add selection to clicked row
    row.classList.add('selected');
    
    const index = parseInt(row.dataset.index);
    selectedMember = filteredMembers[index];
    
    // Enable select button
    document.getElementById('selectButton').disabled = false;
}

// Search/filter members
function searchMembers(searchTerm) {
    const term = searchTerm.toLowerCase().trim();
    
    if (term === '') {
        filteredMembers = [...allMembers];
    } else {
        filteredMembers = allMembers.filter(member => {
            return (
                (member.First_Name || '').toLowerCase().includes(term) ||
                (member.Last_Name || '').toLowerCase().includes(term) ||
                (member.Address || '').toLowerCase().includes(term) ||
                (member.Customer_Card_Number || '').toLowerCase().includes(term)
            );
        });
    }
    
    selectedMember = null;
    document.getElementById('selectButton').disabled = true;
    renderTable(filteredMembers);
}

// Escape HTML to prevent XSS
function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

// Event Listeners
document.getElementById('searchField').addEventListener('input', (e) => {
    searchMembers(e.target.value);
});

document.getElementById('searchField').addEventListener('click', function() {
    this.select();
});

document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
        document.getElementById('searchField').value = '';
        searchMembers('');
        document.getElementById('searchField').focus();
    }
});

document.getElementById('cancelButton').addEventListener('click', () => {
    // Handle cancel action (close window, navigate back, etc.)
    window.close();
});

document.getElementById('selectButton').addEventListener('click', () => {
    if (selectedMember) {
        console.log('Selected member:', selectedMember);
        
        // The selectedMember object contains ALL member data including the hidden ID
        // You can access it like this:
        const memberId = selectedMember.ID;
        const cardNumber = selectedMember.Customer_Card_Number;
        
        console.log('Member ID (for backend):', memberId);
        console.log('Card Number:', cardNumber);
        
        // Example: Send to backend or store in localStorage
        // fetch(`/getMember?id=${memberId}`)...
        // OR
        // localStorage.setItem('selectedMemberId', memberId);
        // localStorage.setItem('selectedMember', JSON.stringify(selectedMember));
        
        alert(`Selected: ${selectedMember.First_Name} ${selectedMember.Last_Name}\nCard: ${cardNumber}\nID: ${memberId}`);
    }
});

// Load members on page load
window.addEventListener('DOMContentLoaded', loadMembers);
```

## Key Features

### 1. Data Management
- **allMembers**: Stores the complete list of members fetched from the server
- **filteredMembers**: Contains the current filtered/searched subset
- **selectedMember**: Holds the currently selected member object

### 2. Functions

#### `loadMembers()`
Asynchronously fetches member data from the `../../searchmembers` endpoint and initializes the table.

#### `renderTable(members)`
Dynamically generates HTML table rows from the members array with XSS protection.

#### `selectRow(row)`
Handles row selection, highlighting the selected row and enabling the select button.

#### `searchMembers(searchTerm)`
Filters members based on search term across all visible fields (First Name, Last Name, Address, Card Number).

#### `escapeHtml(text)`
Sanitizes text to prevent XSS attacks.

### 3. Event Handlers
- **Search input**: Real-time filtering as user types
- **Search field click**: Auto-selects text for easy clearing
- **Escape key**: Clears search and refocuses search field
- **Cancel button**: Closes the window
- **Select button**: Processes the selected member (logs data, shows alert)

## Data Structure

Each member object contains:
- `ID` - Hidden identifier for backend operations
- `First_Name` - Member's first name
- `Last_Name` - Member's last name
- `Address` - Member's address
- `Customer_Card_Number` - Member's card number

## Usage Notes

- The ID field is stored in the data but not displayed in the table
- Selected member data (including ID) is accessible via the `selectedMember` variable
- The code includes XSS protection through HTML escaping
- The select button is disabled until a row is selected

## Dependencies

Requires HTML elements with the following IDs:
- `tableBody` - Table body element for rendering rows
- `searchField` - Input field for search functionality
- `selectButton` - Button to confirm selection
- `cancelButton` - Button to cancel operation