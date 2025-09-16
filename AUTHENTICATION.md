# CoachCare Authentication & Authorization System

This document provides an in-depth explanation of the **Role-Based Access Control (RBAC)** system in the CoachCare platform, focusing on how user roles, permissions, and access control are implemented across the healthcare application.

The document is structured in two main sections:
- **Definitions**: Core concepts, architecture, and system structure
- **Examples**: Practical implementations, code samples, and real-world scenarios

# Definitions

## Overview

The CoachCare platform implements a comprehensive **Role-Based Access Control (RBAC)** system that governs user access to different features, data, and functionality based on their assigned roles within the healthcare ecosystem. The system supports multiple user types from administrators to patients, each with specific permissions tailored to their healthcare responsibilities.

## RBAC Architecture

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│   User Roles        │    │   Permissions       │    │   Access Control    │
│   (Healthcare       │◄──►│   (Feature-based)   │◄──►│   (UI & API)        │
│    Hierarchy)       │    │                     │    │                     │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
        │                           │                           │
        ▼                           ▼                           ▼
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│   13 Defined        │    │   Component-Level   │    │   Route & View      │
│   Role Types        │    │   Permissions       │    │   Protection        │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

## Key Components

1. **Role Hierarchy**: Healthcare-focused role definitions with inheritance
2. **Permission System**: Granular feature and component-level access control
3. **Access Control Engine**: Runtime permission checking and UI rendering
4. **Route Protection**: Page and component-level access restrictions
5. **Dynamic Permissions**: Context-aware permission management

## Role Hierarchy System

### Healthcare Role Structure

The CoachCare RBAC system defines 13 distinct roles organized in a healthcare-focused hierarchy:

#### Administrative Roles
```javascript
const authRoles = {
  admin: {
    id: "ADMIN",
    roles: ["ADMIN"],
    title: "System Administrator"
  },
  sentinelAdmin: {
    id: "SENTINEL_ADMIN",
    roles: ["ADMIN", "SENTINEL_ADMIN"],
    title: "CoachCare Administrator"
  },
  sentinelStaff: {
    id: "SENTINEL_STAFF",
    roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF"],
    title: "CoachCare Staff"
  },
  sentinelDeveloper: {
    id: "SENTINEL_DEVELOPER",
    roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_DEVELOPER"],
    title: "Platform Developer"
  }
};
```

#### Clinical Roles
```javascript
clinicAdminPlus: {
  id: "CLINIC_ADMIN_PLUS",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "CLINIC_ADMIN_PLUS"],
  title: "Enhanced Clinic Administrator"
},
clinicAdmin: {
  id: "CLINIC_ADMIN",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "CLINIC_ADMIN_PLUS", "CLINIC_ADMIN"],
  title: "Clinic Administrator"
},
clinicStaff: {
  id: "CLINIC_STAFF",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "CLINIC_ADMIN", "CLINIC_ADMIN_PLUS", "PROVIDER", "CLINIC_STAFF"],
  title: "Clinic Staff Member"
},
provider: {
  id: "PROVIDER",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "CLINIC_ADMIN", "CLINIC_ADMIN_PLUS", "PROVIDER"],
  title: "Healthcare Provider"
}
```

#### Specialized Roles
```javascript
// Supplemental roles for specific responsibilities
medicationRequestAdmin: {
  id: "MEDICATION_REQUEST_ADMIN",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "MEDICATION_REQUEST_ADMIN"],
  title: "Medication Request Administrator"
},
careTeamManager: {
  id: "CARE_TEAM_MANAGER",
  roles: ["ADMIN", "SENTINEL_ADMIN"],
  title: "Care Team Manager"
},
careTeamOwner: {
  id: "CARE_TEAM_OWNER",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF", "CARE_TEAM_OWNER"],
  title: "Care Team Owner"
},
billingManager: {
  id: "BILLING_MANAGER",
  roles: ["ADMIN", "SENTINEL_ADMIN", "SENTINEL_STAFF"],
  title: "Billing Manager"
}
```

#### End User Roles
```javascript
patient: {
  id: "PATIENT",
  roles: ["PATIENT"],
  title: "Patient"
}
```

### Role Inheritance Pattern

The role system implements hierarchical inheritance where higher-level roles automatically inherit permissions from lower-level roles:

Role hierarchy (highest to lowest privilege):
ADMIN → SENTINEL_ADMIN → SENTINEL_STAFF → CLINIC_ADMIN_PLUS → CLINIC_ADMIN → PROVIDER → CLINIC_STAFF → PATIENT

### Role Validation Functions

```javascript
// Check if user has specific role access
export const userHasRole = (user, authRole) => {
  return (
    user &&
    user.roles &&
    authRole.roles.find((r) => user.roles.includes(r)) !== undefined
  );
};

// Generate appropriate role list for user management
export const generateRoleList = (user) => {
  const [authorization] = user.roles;
  const authIndex = allProviderRoles.findIndex((role) => role.id === authorization);

  if (userHasRole(user, authRoles.admin)) {
    return allProviderRoles; // Admin can assign any role
  } else if (authorization === "PROVIDER") {
    return allProviderRoles.slice(authIndex - 1); // Providers can manage staff and below
  } else {
    return allProviderRoles.slice(authIndex); // Others can only manage same level and below
  }
};
```

## Permission System Implementation

### Component-Level Permissions

The system implements granular permissions for UI components and features:

#### Permission Structure
```javascript
// Example: Admin permissions for patient management
export default {
  patients: {
    components: {
      patientClinicalHeader: {
        views: [
          { label: "Clinical", name: "clinical" },
          { label: "Billing", name: "billing" },
          { label: "COVID", name: "covid" }
        ],
        allowAddCovid: true,
        allowAdd: true
      }
    },
    views: [
      { label: "Clinical", name: "clinical" },
      { label: "Billing", name: "billing" },
      { label: "COVID", name: "covid" }
    ]
  },
  patient: {
    components: {
      profileInfo: {
        allowEdit: true,
        allowEditProfile: true,
        allowEditPassword: true
      },
      patientInfoHeader: {
        allowCoBrowse: true,
        allowChatLog: true,
        allowVitals: true,
        allowNote: true,
        allowMedication: true,
        allowLabs: true,
        allowCognitiveTests: true,
        allowWorkList: true,
        allowContacts: true,
        allowPatientForms: true
      },
      patientEditHeader: {
        allowComplianceDate: true,
        allowInsuranceInfo: true,
        allowCaseInvestigation: true,
        allowChronicDiseases: true,
        allowTeamMembers: true,
        allowBillingCodes: true
      }
    },
    showCharts: true,
    showVitals: true,
    showMedications: true,
    showSymptoms: true,
    showNotes: true,
    infoForms: ["personal", "medical", "insurance"]
  }
};
```

Permission Inheritance & Combination

```javascript
// Combine multiple role permissions
export const combineRolesAccess = (access, page) => {
  const result = Object.assign({}, defaultAccess[page]);

  Object.keys(access).forEach((roleKey) => {
    Object.keys(access[roleKey]).forEach((attributeKey) => {
      if (typeof access[roleKey][attributeKey] === "boolean") {
        // Boolean OR - grant access if any role allows it
        result[attributeKey] = result[attributeKey] || access[roleKey][attributeKey];
      } else if (Array.isArray(access[roleKey][attributeKey])) {
        // Array merge - combine unique values from all roles
        result[attributeKey] = [
          ...new Set(
            result[attributeKey]
              .concat(access[roleKey][attributeKey])
              .map(JSON.stringify)
          ),
        ].map(JSON.parse);
      } else if (attributeKey === "components") {
        // Component-specific role mapping
        Object.keys(result[attributeKey]).forEach((componentKey) => {
          result[attributeKey][componentKey][roleKey] =
            access[roleKey][attributeKey][componentKey];
        });
      }
    });
  });

  return result;
};
```

Role-Based UI Rendering

Navigation Access Control
```javascript
// Navigation component checks user role before rendering
const { item, nestedLevel, classes, userRole, active } = this.props;

// Hide navigation items user doesn't have access to
if (
  item.auth &&
  (!item.auth.includes(userRole) ||
    (userRole !== "unauthorized" &&
     !item.auth.includes(userRole)))
) {
  return null; // Don't render this navigation item
}
```

Dynamic Permission Checking

```javascript
// Real-time permission validation in components
const PatientComponent = ({ user, permissions }) => {
  const canEditPatient = permissions.patient?.components?.profileInfo?.allowEdit;
  const canViewBilling = permissions.patients?.views?.some(v => v.name === 'billing');
  const availableViews = permissions.patients?.views || [];

  return (
    <div>
      {availableViews.map(view => (
        <Tab key={view.name} label={view.label} />
      ))}

      {canEditPatient && (
        <Button onClick={handleEdit}>Edit Patient</Button>
      )}

      {canViewBilling && (
        <BillingSection />
      )}
    </div>
  );
};
```

Access Control Implementation

Route Protection

The system implements route-level access control through authentication guards:

```javascript
// Route configuration with role-based access
const routes = [
  {
    path: '/patients',
    component: PatientsPage,
    auth: {
      roles: ['ADMIN', 'SENTINEL_ADMIN', 'SENTINEL_STAFF', 'CLINIC_ADMIN', 'PROVIDER']
    }
  },
  {
    path: '/billing',
    component: BillingPage,
    auth: {
      roles: ['ADMIN', 'SENTINEL_ADMIN', 'BILLING_MANAGER']
    }
  },
  {
    path: '/patient/:id/edit',
    component: PatientEdit,
    auth: {
      roles: ['ADMIN', 'SENTINEL_ADMIN', 'SENTINEL_STAFF', 'CLINIC_ADMIN']
    }
  }
];

// Route access validation
static setRoutes(config) {
  let routes = [...config.routes];

  if (config.auth) {
    routes = routes.map((route) => {
      let auth = config.auth && config.auth.roles ? [...config.auth.roles] : [];
      auth = route.auth ? [...auth, ...route.auth.roles] : auth;
      return {
        ...route,
        auth
      };
    });
  }

  return [...routes];
}
```

State Management Integration

User Role State
```javascript
// Redux state for user roles and permissions
const initialState = {
  roles: [authRoles.onlyUnauthorized.id], // Default unauthorized
  permissions: [],
  providerClinics: [],
  patientClinics: [],
  isLoggedIn: false
};

const user = function (state = initialState, action) {
  switch (action.type) {
    case Actions.SET_USER_DATA: {
      return {
        ...initialState,
        ...action.payload, // Contains roles, permissions, clinic access
        isLoggedIn: true,
      };
    }
    case Actions.USER_LOGGED_OUT: {
      return { ...initialState }; // Reset to unauthorized
    }
  }
};
```

Permission State Management
```javascript
// Permissions reducer managing access control
const initialState = {
  access: {
    UNAUTHORIZED: defaultAccess,
    ADMIN: defaultAccess,
    SENTINEL_ADMIN: defaultAccess,
    SENTINEL_STAFF: defaultAccess,
    CLINIC_ADMIN: defaultAccess,
    CLINIC_STAFF: defaultAccess,
    PROVIDER: defaultAccess,
    PATIENT: defaultAccess,
    SENTINEL_DEVELOPER: defaultAccess,
  },
};

const permissions = function (state = initialState, action) {
  switch (action.type) {
    case Actions.FETCH_PERMISSIONS: {
      return {
        ...state,
        access: action.payload, // Dynamic permissions loaded from backend
      };
    }
  }
};
```

# Examples

## Healthcare Workflow Scenarios

Scenario 1: Patient Data Access
```javascript
// Admin can access all patient data and billing information
const adminAccess = {
  patient: {
    showCharts: true,
    showVitals: true,
    showMedications: true,
    showSymptoms: true,
    showNotes: true,
    components: {
      patientInfoHeader: {
        allowCoBrowse: true,
        allowChatLog: true,
        allowVitals: true,
        allowNote: true,
        allowMedication: true,
        allowLabs: true,
        allowBillingCodes: true
      }
    }
  }
};

// Provider has clinical access but limited administrative functions
const providerAccess = {
  patient: {
    showCharts: true,
    showVitals: true,
    showMedications: true,
    showSymptoms: true,
    showNotes: true,
    components: {
      patientInfoHeader: {
        allowVitals: true,
        allowNote: true,
        allowMedication: true,
        allowLabs: true,
        allowBillingCodes: false // No billing access
      }
    }
  }
};

// Patient has read-only access to their own data
const patientAccess = {
  patient: {
    showCharts: true,
    showVitals: true,
    showMedications: true,
    showSymptoms: false, // Cannot view sensitive symptoms
    showNotes: false,    // Cannot view provider notes
    components: {
      profileInfo: {
        allowEditProfile: true,
        allowEditPassword: true,
        allowEdit: false // Cannot edit medical data
      }
    }
  }
};
```

Scenario 2: Supplemental Role Management
```javascript
// Users who can assign supplemental roles
export const suppRolesAccessedBy = [
  authRoles.admin.id,           // System admin
  authRoles.sentinelAdmin.id,   // CoachCare admin
  authRoles.sentinelStaff.id,   // CoachCare staff
];

// Specialized role assignment - RRT Manager
export const rrtManagerRoleAccessedBy = [
  authRoles.admin.id,
  authRoles.sentinelAdmin.id,
  authRoles.sentinelStaff.id,
  authRoles.clinicAdminPlus.id, // Enhanced clinic admin can assign
  authRoles.clinicAdmin.id,     // Regular clinic admin can assign
];

// Example: Provider with additional billing responsibilities
const providerWithBilling = {
  primaryRole: "PROVIDER",
  supplementalRoles: ["BILLING_MANAGER"],
  permissions: {
    // Inherits all provider permissions PLUS billing access
    ...providerAccess,
    billing: {
      allowViewReports: true,
      allowGenerateInvoices: true,
      allowManageCodes: true
    }
  }
};
```

Component Integration Examples

Protected Navigation Menu
```javascript
const NavigationMenu = ({ user, userRole }) => {
  return (
    <nav>
      {/* Always visible */}
      <MenuItem to="/dashboard">Dashboard</MenuItem>

      {/* Admin and management roles only */}
      {userHasRole(user, authRoles.sentinelAdmin) && (
        <MenuItem to="/system-admin">System Administration</MenuItem>
      )}

      {/* Clinical staff and above */}
      {userHasRole(user, authRoles.provider) && (
        <MenuItem to="/patients">Patient Management</MenuItem>
      )}

      {/* Billing managers only */}
      {userHasRole(user, authRoles.billingManager) && (
        <MenuItem to="/billing">Billing & Reports</MenuItem>
      )}

      {/* Medication request administrators */}
      {userHasRole(user, authRoles.medicationRequestAdmin) && (
        <MenuItem to="/medications">Medication Requests</MenuItem>
      )}

      {/* Patients only */}
      {userRole === "PATIENT" && (
        <MenuItem to="/my-health">My Health Data</MenuItem>
      )}
    </nav>
  );
};
```

Dynamic Form Permissions
```javascript
const PatientEditForm = ({ user, permissions, patient }) => {
  // Get role-specific permissions for patient editing
  const editPermissions = permissions.patient?.components?.patientEditHeader || {};

  return (
    <form>
      {/* Basic information - all clinical roles */}
      <BasicInfoSection />

      {/* Insurance info - clinic admin and above */}
      {editPermissions.allowInsuranceInfo && (
        <InsuranceSection patient={patient} />
      )}

      {/* Billing codes - admin and billing managers only */}
      {editPermissions.allowBillingCodes && (
        <BillingCodesSection patient={patient} />
      )}

      {/* Case investigation - sentinel staff and above */}
      {editPermissions.allowCaseInvestigation && (
        <CaseInvestigationSection patient={patient} />
      )}

      {/* Team member assignment - clinic admin plus and above */}
      {editPermissions.allowTeamMembers && userHasRole(user, authRoles.clinicAdminPlus) && (
        <TeamMembersSection patient={patient} />
      )}

      {/* Compliance dates - specialized access */}
      {editPermissions.allowComplianceDate && (
        <ComplianceDateSection patient={patient} />
      )}
    </form>
  );
};
```

Authentication Integration

Login Flow with RBAC
```javascript
// After successful Cognito authentication, load user roles and permissions
export function submitLogin({ email, password }) {
  return (dispatch) => {
    return new Promise((resolve, reject) => {
      const handleSuccess = (user) => {
        // Set user data including roles
        dispatch(setUserData(user)); // Contains: { roles: ["PROVIDER"], permissions: [...] }

        // Track login with role information
        AppInsightsService.setGlobalProperties({ roles: user.roles });
        AppInsightsService.eventTracker("User Login Success");

        // Load role-specific permissions
        dispatch(fetchPermissions());

        // Redirect based on user role
        const redirectPath = determineHomePage(user.roles[0]);
        history.push(redirectPath);

        resolve(user);
      };

      cognitoService.signInWithUsernameAndPassword(email, password, handleSuccess, handleError);
    });
  };
}

// Determine home page based on primary role
const determineHomePage = (primaryRole) => {
  switch (primaryRole) {
    case 'ADMIN':
    case 'SENTINEL_ADMIN':
      return '/system-dashboard';
    case 'SENTINEL_STAFF':
      return '/operations-dashboard';
    case 'CLINIC_ADMIN':
    case 'CLINIC_ADMIN_PLUS':
      return '/clinic-dashboard';
    case 'PROVIDER':
      return '/patient-dashboard';
    case 'CLINIC_STAFF':
      return '/work-lists';
    case 'PATIENT':
      return '/my-dashboard';
    default:
      return '/dashboard';
  }
};
```

Token-Based Role Verification
```javascript
// Backend API validates user tokens and returns role-specific data
const requestInterceptor = (req) => {
  return new Promise((resolve, reject) => {
    const currentUser = this.userPool.getCurrentUser();

    if (!currentUser) {
      this.emit("onAutoLogout", "user not found");
      reject("no user found");
    } else {
      currentUser.getSession((err, session) => {
        if (err) {
          this.emit("onAutoLogout", "session expired");
          reject(err);
        } else {
          // Attach JWT token with embedded role information
          this.usertoken = session.getAccessToken().getJwtToken();
          req.headers.Authorization = `Bearer ${this.usertoken}`;

          // Token contains user roles for backend RBAC validation
          resolve(req);
        }
      });
    }
  });
};
```

Backend RBAC Integration

API-Level Role Checking
```javascript
// Lambda authorizer validates roles from JWT token
const authorizeRequest = (event, context) => {
  const token = extractTokenFromEvent(event);
  const decodedToken = jwt.verify(token, secret);

  // Extract roles from token claims
  const userRoles = decodedToken['custom:roles'] || [];
  const requiredRoles = event.pathParameters.requiredRoles || [];

  // Check if user has any of the required roles
  const hasAccess = requiredRoles.some(role =>
    authRoles[role].roles.some(r => userRoles.includes(r))
  );

  if (!hasAccess) {
    return generatePolicy('user', 'Deny', event.methodArn);
  }

  return generatePolicy('user', 'Allow', event.methodArn);
};

// Example API Gateway configuration
const apiRoutes = {
  'GET /patients': { roles: ['PROVIDER', 'CLINIC_ADMIN', 'SENTINEL_STAFF'] },
  'POST /patients': { roles: ['CLINIC_ADMIN', 'SENTINEL_STAFF'] },
  'PUT /patients/{id}/billing': { roles: ['BILLING_MANAGER', 'ADMIN'] },
  'GET /system/users': { roles: ['ADMIN', 'SENTINEL_ADMIN'] },
  'POST /medications/requests': { roles: ['MEDICATION_REQUEST_ADMIN', 'PROVIDER'] }
};
```

Security Features

### Hierarchical Role Inheritance
- **Automatic Permission Inheritance**: Higher roles inherit lower-level permissions
- **Principle of Least Privilege**: Users get minimum necessary access
- **Role-Based UI Rendering**: Components only render if user has access

### Dynamic Permission Loading
- **Runtime Permission Checks**: Permissions validated in real-time
- **Context-Aware Access**: Permissions can vary by clinic, patient, or context
- **Fallback to Defaults**: Secure fallback when permissions can't be loaded

### Audit Trail & Compliance
- **Role Assignment Tracking**: All role changes logged
- **Access Attempt Logging**: Failed access attempts recorded
- **Compliance Reports**: Healthcare compliance role reporting

## Best Practices for RBAC Implementation

### Role Design Principles
- **Healthcare-Focused**: Roles reflect real healthcare organizational structure
- **Granular Permissions**: Fine-grained control over features and data
- **Separation of Duties**: Billing, clinical, and administrative roles separated
- **Supplemental Roles**: Additional responsibilities without changing primary role

### Implementation Guidelines
- **Fail Secure**: Default to no access when uncertain
- **Regular Audits**: Periodic review of role assignments
- **Documentation**: Clear documentation of role capabilities
- **Testing**: Comprehensive testing of role-based access scenarios

### User Management
- **Role Assignment Workflow**: Structured process for assigning roles
- **Temporary Access**: Time-limited role assignments for specific tasks
- **Role Change Notifications**: Notify users when roles are modified
- **Self-Service Options**: Allow users to request role changes through proper channels

### Monitoring & Maintenance
- **Access Pattern Analysis**: Monitor role usage patterns
- **Permission Creep Prevention**: Regular cleanup of unnecessary permissions
- **Role Effectiveness**: Measure if roles meet business needs
- **Security Incident Response**: Quick role modification in case of security issues

## Development & Testing

### Role-Based Testing Scenarios
```javascript
// Test helper for role-based component testing
const renderWithRole = (component, userRole = 'UNAUTHORIZED') => {
  const mockUser = {
    roles: [userRole],
    permissions: permissions[userRole] || defaultAccess
  };

  const mockStore = configureMockStore([])({
    auth: { user: mockUser },
    permissions: { access: permissions }
  });

  return render(
    <Provider store={mockStore}>
      {component}
    </Provider>
  );
};

// Example test cases
describe('Patient Component RBAC', () => {
  it('should show edit button for clinic admin', () => {
    renderWithRole(<PatientDetails />, 'CLINIC_ADMIN');
    expect(screen.getByText('Edit Patient')).toBeInTheDocument();
  });

  it('should hide billing section for providers', () => {
    renderWithRole(<PatientDetails />, 'PROVIDER');
    expect(screen.queryByText('Billing Information')).not.toBeInTheDocument();
  });

  it('should show all sections for admin', () => {
    renderWithRole(<PatientDetails />, 'ADMIN');
    expect(screen.getByText('Billing Information')).toBeInTheDocument();
    expect(screen.getByText('Edit Patient')).toBeInTheDocument();
    expect(screen.getByText('Clinical Data')).toBeInTheDocument();
  });
});
```

This comprehensive RBAC system ensures that healthcare data and functionality are appropriately secured while providing the flexibility needed for complex healthcare workflows and organizational structures.