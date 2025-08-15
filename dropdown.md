# Storing and Retrieving Lead Owner in Sequelize

## 1. Storing the Lead Owner

Your `Lead` model already has `ownerId` referencing `users.id`.\
When creating a lead, pass the selected user's ID into the `ownerId`
field.

**Example Controller (Create Lead)**

``` ts
// controllers/leadController.ts
import Lead from '../models/Lead';

export const createLead = async (req, res) => {
  try {
    const { firstName, lastName, email, phone, ownerId, jobTitle, leadStatus } = req.body;

    const lead = await Lead.create({
      firstName,
      lastName,
      email,
      phone,
      ownerId,
      jobTitle,
      leadStatus
    });

    res.status(201).json(lead);
  } catch (error) {
    console.error('Error creating lead:', error);
    res.status(500).json({ error: 'Failed to create lead' });
  }
};
```

**Frontend form submission**

``` ts
const payload = {
  firstName,
  lastName,
  email,
  phone,
  ownerId: selectedOwnerId,
  jobTitle,
  leadStatus,
};

await api.post('/leads', payload);
```

## 2. Retrieving Leads With Owner Name

Use Sequelize's `include` to join the `User` table.

``` ts
import Lead from '../models/Lead';
import User from '../models/User';

export const getAllLeads = async (req, res) => {
  try {
    const leads = await Lead.findAll({
      include: [
        {
          model: User,
          as: 'owner',
          attributes: ['id', 'name'],
        },
      ],
    });

    res.json(leads);
  } catch (error) {
    console.error('Error fetching leads:', error);
    res.status(500).json({ error: 'Failed to fetch leads' });
  }
};
```

**Example Response**

``` json
[
  {
    "id": 1,
    "firstName": "John",
    "lastName": "Smith",
    "email": "john@example.com",
    "phone": "1234567890",
    "ownerId": 2,
    "owner": {
      "id": 2,
      "name": "Alice Johnson"
    }
  }
]
```

**Frontend display**

``` tsx
{leads.map(lead => (
  <tr key={lead.id}>
    <td>{lead.firstName} {lead.lastName}</td>
    <td>{lead.email}</td>
    <td>{lead.phone}</td>
    <td>{lead.owner?.name || 'No Owner'}</td>
  </tr>
))}
```

------------------------------------------------------------------------

# Implementing a Dynamic Dropdown in Create Lead Form

## 1. Fetch Users From API

Import your `useSortedUsers` hook.

``` ts
import { useSortedUsers } from '../../hooks/useSortedUsers';
```

## 2. Map Users to Dropdown Options

``` ts
const { users, isLoading } = useSortedUsers();

const ownerOptions = users.map(user => ({
  label: user.name,
  value: user.id
}));
```

## 3. Store Owner ID in State

``` ts
contactOwner: '', // store ownerId instead of name
```

## 4. Contact Owner Dropdown Component

``` tsx
<label className="text-xs">Contact Owner</label>
<DropDown
  id="contactOwner"
  label="Choose"
  value={leadData.contactOwner}
  onChange={(e) =>
    setLeadData((prev) => ({
      ...prev,
      contactOwner: Number(e.target.value)
    }))
  }
  options={ownerOptions}
  className="w-full border border-gray-300 rounded px-3 py-2"
/>
{isLoading && <p className="text-gray-500 text-xs">Loading users...</p>}
```

## 5. Send Owner ID to Backend

``` ts
const payload = {
  firstName: leadData.firstName,
  lastName: leadData.lastName,
  email: leadData.email,
  phone: leadData.phone,
  ownerId: leadData.contactOwner,
  jobTitle: leadData.jobTitle,
  leadStatus: leadData.leadStatus,
  createdAt: new Date().toISOString()
};
```

**Flow:** 1. Fetch all users with `useSortedUsers()`. 2. Populate
dropdown dynamically. 3. Store selected `ownerId`. 4. Send `ownerId` to
backend when creating lead. 5. Backend saves `ownerId` and relates it to
`User`.
