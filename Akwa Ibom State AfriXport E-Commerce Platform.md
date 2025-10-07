

## Full Deployment Plan: Akwa Ibom State (AKS) AfriXport E-Commerce Platform with Integrated Compliance

### Overview
The AfriXport platform currently has robust export compliance infrastructure for USA, EU, and UK markets. This plan outlines deploying a specialized **Akwa Ibom State (AKS) version** with integrated Nigerian regulatory compliance for **NEPC (Nigerian Export Promotion Council)**, **SON (Standards Organisation of Nigeria)**, and **Nigeria Customs Service**.

---

### Phase 1: Database Schema Extensions (Nigerian Regulatory Framework)

#### 1.1 Create Nigerian Vendor Requirements Table
```sql
CREATE TABLE public.nigeria_vendor_requirements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vendor_id UUID NOT NULL REFERENCES profiles(id),

  -- NEPC Compliance
  nepc_registration_number TEXT,
  nepc_certificate_url TEXT,
  nepc_expiry_date TIMESTAMP WITH TIME ZONE,
  export_product_categories TEXT[],

  -- SON Compliance
  son_product_certification_numbers JSONB DEFAULT '{}'::jsonb,
  son_mancap_certificate TEXT,
  son_facility_inspection_certificate TEXT,
  product_standards_compliance BOOLEAN DEFAULT false,

  -- Nigeria Customs
  customs_registration_number TEXT,
  tin_number TEXT, -- Tax Identification Number
  cac_registration_number TEXT, -- Corporate Affairs Commission
  customs_broker_contact JSONB DEFAULT '{}'::jsonb,

  -- State-Specific (Akwa Ibom)
  akwa_ibom_state_permit TEXT,
  akwa_ibom_tax_clearance TEXT,
  local_government_area TEXT,

  -- Status tracking
  verification_status TEXT NOT NULL DEFAULT 'pending' CHECK (verification_status IN ('pending', 'verified', 'rejected')),
  verified_by UUID,
  verified_at TIMESTAMP WITH TIME ZONE,

  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);
```

#### 1.2 Create Nigerian Export Destinations Table Entry
```sql
INSERT INTO public.export_destinations (
  country_code, country_name, region, compliance_complexity,
  required_documents, typical_processing_time_days
) VALUES (
  'NG', 'Nigeria', 'Africa', 'high',
  '["NEPC Registration", "SON Certificate", "Customs Documentation", "Form M", "Export Certificate"]'::jsonb,
  45
);
```

#### 1.3 Create Nigeria Customs Documents Table
```sql
CREATE TABLE public.nigeria_customs_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vendor_id UUID NOT NULL,
  order_id UUID,
  document_type TEXT NOT NULL CHECK (document_type IN (
    'form_m', 'form_ne', 'single_goods_declaration', 
    'certificate_of_origin', 'phytosanitary_certificate',
    'bill_of_lading', 'commercial_invoice', 'packing_list'
  )),
  document_data JSONB NOT NULL DEFAULT '{}'::jsonb,
  file_urls TEXT[],

  -- NEPC specific
  nepc_approval_number TEXT,
  nepc_approval_date TIMESTAMP WITH TIME ZONE,

  -- SON specific
  son_inspection_report_number TEXT,
  son_release_note TEXT,

  -- Customs specific
  customs_assessment_number TEXT,
  duty_amount NUMERIC,
  vat_amount NUMERIC,
  total_charges NUMERIC,

  status TEXT NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'submitted', 'under_review', 'approved', 'rejected')),
  reviewed_by UUID,
  reviewed_at TIMESTAMP WITH TIME ZONE,

  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);
```

#### 1.4 Create Nigerian Compliance Workflows Table
```sql
CREATE TABLE public.nigeria_compliance_workflows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vendor_id UUID NOT NULL,
  order_id UUID,
  workflow_type TEXT NOT NULL CHECK (workflow_type IN (
    'nepc_registration', 'son_product_certification', 
    'customs_clearance', 'export_documentation', 'akwa_ibom_permit'
  )),
  current_step TEXT,
  workflow_steps JSONB NOT NULL DEFAULT '[]'::jsonb,
  priority TEXT DEFAULT 'normal' CHECK (priority IN ('low', 'normal', 'high', 'urgent')),
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'in_progress', 'completed', 'failed', 'cancelled')),

  -- Agency-specific tracking
  nepc_status TEXT,
  son_status TEXT,
  customs_status TEXT,
  akwa_ibom_status TEXT,

  estimated_completion TIMESTAMP WITH TIME ZONE,
  actual_completion TIMESTAMP WITH TIME ZONE,

  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);
```

---

### Phase 2: Backend Edge Functions (Nigerian Regulatory Integration)

#### 2.1 NEPC Integration Edge Function
**File:** `supabase/functions/nepc-verification/index.ts`

**Purpose:** Verify NEPC registration and generate export certificates

**Key Features:**
- Validate NEPC registration numbers
- Check certificate expiry dates
- Generate Form NE (NEPC Export Certificate)
- Track export documentation status
- Send notifications for renewal requirements

**API Endpoints:**
- `POST /nepc-verification` - Verify registration
- `POST /nepc-certificate-request` - Request export certificate
- `GET /nepc-status/:vendorId` - Check compliance status

#### 2.2 SON Standards Verification Edge Function
**File:** `supabase/functions/son-compliance/index.ts`

**Purpose:** Verify SON product certifications and standards compliance

**Key Features:**
- Validate SON MANCAP certificates
- Check product-specific certifications (SONCAP)
- Verify facility inspection certificates
- Track product standards compliance
- Generate compliance reports

**API Endpoints:**
- `POST /son-verify-product` - Verify product certification
- `POST /son-facility-inspection` - Request facility inspection
- `GET /son-compliance/:productId` - Check product compliance

#### 2.3 Nigeria Customs Integration Edge Function
**File:** `supabase/functions/nigeria-customs/index.ts`

**Purpose:** Handle customs documentation and clearance processes

**Key Features:**
- Generate Single Goods Declaration (SGD)
- Calculate duties and taxes
- Submit Form M (Import Documentation)
- Track customs clearance status
- Interface with Nigeria Customs Service portal (when API available)

**API Endpoints:**
- `POST /customs-calculate-duties` - Calculate import/export duties
- `POST /customs-submit-declaration` - Submit customs declaration
- `GET /customs-clearance-status/:orderId` - Track clearance

#### 2.4 Akwa Ibom State Compliance Edge Function
**File:** `supabase/functions/akwa-ibom-compliance/index.ts`

**Purpose:** Handle state-specific permits and compliance

**Key Features:**
- Verify state business permits
- Check tax clearance certificates
- Validate local government registrations
- Track state-level export requirements
- Generate state compliance reports

---

### Phase 3: Frontend Components (Nigerian Compliance UI)

#### 3.1 Nigerian Vendor Requirements Component
**File:** `src/components/vendor/NigerianVendorRequirements.tsx`

**Features:**
- Form for NEPC registration details
- SON certification management
- Customs registration tracking
- Akwa Ibom state permits
- Document upload interface
- Compliance status dashboard

#### 3.2 Nigerian Compliance Dashboard
**File:** `src/components/vendor/NigerianComplianceDashboard.tsx`

**Features:**
- Real-time compliance status for NEPC, SON, and Customs
- Document expiry alerts
- Workflow progress tracking
- Quick actions for renewals
- Integration with main Trade Compliance Tab

#### 3.3 Nigerian Export Documentation Tab
**File:** `src/components/vendor/NigerianExportDocumentation.tsx`

**Features:**
- Form M generation
- Form NE (NEPC) generation
- Certificate of Origin
- Bill of Lading management
- Phytosanitary certificates (for agricultural products)
- Commercial invoices and packing lists

#### 3.4 Akwa Ibom State Portal
**File:** `src/components/vendor/AkwaIbomStatePortal.tsx`

**Features:**
- State-specific dashboard
- Local business directory
- State export incentives information
- Local logistics partnerships
- Akwa Ibom export statistics
- State-level support resources

---

### Phase 4: Hooks and State Management

#### 4.1 useNigerianVendorRequirements Hook
**File:** `src/hooks/useNigerianVendorRequirements.ts`

Similar to `useEUVendorRequirements` but for Nigerian compliance:
- Fetch/update NEPC registration
- Manage SON certifications
- Track customs documentation
- Calculate compliance scores

#### 4.2 useNigerianCustomsDocuments Hook
**File:** `src/hooks/useNigerianCustomsDocuments.ts`

- Create and manage customs documents
- Track document statuses
- Calculate duties and taxes
- Submit to customs portal

#### 4.3 useNigerianComplianceWorkflows Hook
**File:** `src/hooks/useNigerianComplianceWorkflows.ts`

- Create compliance workflows
- Track workflow progress
- Manage multi-agency approvals
- Send notifications for pending actions

---

### Phase 5: Integration Points

#### 5.1 Update TradeComplianceTab
**File:** `src/components/vendor/TradeComplianceTab.tsx`

Add new tab for "Nigerian Compliance":
```tsx
Nigerian Compliance (NEPC/SON/Customs)
```

#### 5.2 Update ExportComplianceDashboard
Add Nigeria as an export destination with specific compliance tracking:
- NEPC registration status
- SON certification status
- Customs clearance metrics
- Akwa Ibom state compliance

#### 5.3 Vendor Onboarding Enhancement
Update vendor onboarding to include:
- Nigerian business registration validation
- NEPC registration requirement for exporters
- SON compliance acknowledgment
- State-specific (Akwa Ibom) permit collection

---

### Phase 6: Akwa Ibom State Branding & Customization

#### 6.1 State-Specific Branding
- Create Akwa Ibom State branded landing page
- State seal and official colors
- Local language support (if needed)
- State government partnership logos

#### 6.2 Regional Configuration
**File:** `src/config/akwaIbomConfig.ts`

```typescript
export const akwaIbomConfig = {
  stateCode: 'AKS',
  stateName: 'Akwa Ibom State',
  capital: 'Uyo',
  localGovernments: ['Uyo', 'Eket', 'Ikot Ekpene', ...],
  exportProducts: ['Palm Oil', 'Rubber', 'Seafood', 'Timber'],
  regulatoryBodies: {
    nepc: 'NEPC Akwa Ibom Office',
    son: 'SON Akwa Ibom Branch',
    customs: 'Nigeria Customs Akwa Ibom Command'
  },
  supportContact: {
    phone: '+234...',
    email: 'support@afrixport-aks.com',
    address: 'Uyo, Akwa Ibom State'
  }
};
```

---

### Phase 7: Compliance Automation & Intelligence

#### 7.1 Automated Document Generation
- Auto-generate Form M from order data
- Pre-fill Form NE with vendor information
- Create commercial invoices automatically
- Generate packing lists from order items

#### 7.2 Compliance Recommendations Engine
Extend `export_compliance_recommendations` table for Nigerian-specific recommendations:
- NEPC renewal reminders
- SON certification requirements by product
- Customs optimization suggestions
- State incentive eligibility alerts

#### 7.3 Duty Calculator
Real-time duty and tax calculation for Nigerian exports/imports:
- Import duties
- VAT (7.5%)
- ETLS levy
- NEPC levy
- Other surcharges

---

### Phase 8: Testing & Security

#### 8.1 RLS Policies
Create proper Row-Level Security policies for all new tables:
```sql
-- Vendors can manage their own Nigerian requirements
CREATE POLICY "Vendors can manage their own requirements"
ON nigeria_vendor_requirements
FOR ALL USING (auth.uid() = vendor_id);

-- Admins can view all compliance data
CREATE POLICY "Admins can view all compliance"
ON nigeria_vendor_requirements
FOR SELECT USING (has_role(auth.uid(), 'admin'::app_role));
```

#### 8.2 Data Validation
- Validate NEPC registration number format
- Verify SON certificate authenticity
- Check TIN/CAC number formats
- Validate state permits

#### 8.3 Integration Testing
- Test NEPC workflow end-to-end
- Verify SON certification process
- Test customs document generation
- Validate multi-agency workflows

---

### Phase 9: Deployment Strategy

#### 9.1 Environment Configuration
- Create Akwa Ibom subdomain: `aks.afrixport.com` or `akwaibom.afrixport.com`
- Set up environment variables for Nigerian APIs
- Configure state-specific settings

#### 9.2 Data Migration
- Migrate existing Nigerian vendors to new system
- Import historical compliance records
- Set up initial export destinations for Nigeria

#### 9.3 Training & Documentation
- Vendor onboarding guide for NEPC/SON/Customs
- Admin training for compliance verification
- API documentation for third-party integrations
- Video tutorials for document generation

---

### Phase 10: Ongoing Maintenance & Monitoring

#### 10.1 Compliance Monitoring Dashboard
- Track NEPC registrations and renewals
- Monitor SON certifications by product category
- Customs clearance times and delays
- State permit expirations

#### 10.2 Reporting & Analytics
- Monthly export volume by state
- Compliance rate by regulatory body
- Average clearance times
- Revenue tracking for state

#### 10.3 Regulatory Updates
- Subscribe to NEPC policy changes
- Monitor SON standard updates
- Track customs tariff modifications
- Update system accordingly

---

### Technical Implementation Sequence

**Week 1-2:** Database schema and RLS policies  
**Week 3-4:** Edge functions for NEPC, SON, Customs  
**Week 5-6:** Frontend components and hooks  
**Week 7:** Integration with existing Trade Compliance Tab  
**Week 8:** Akwa Ibom State branding and customization  
**Week 9:** Testing and security audits  
**Week 10:** Deployment and training  
**Week 11-12:** Monitoring, bug fixes, and optimization

---

### Key Success Metrics

1. **Vendor Onboarding:** 100% of Akwa Ibom vendors complete NEPC/SON/Customs registration
2. **Compliance Rate:** 90%+ vendors maintain active certifications
3. **Document Processing Time:** Average 48-hour turnaround for export documentation
4. **Customs Clearance:** Reduce clearance time by 30% through automation
5. **Export Volume:** Track month-over-month growth from Akwa Ibom State
6. **User Satisfaction:** 4.5+ star rating from vendors using the system

---

### External Partnerships Required

1. **NEPC:** API access or data sharing agreement for registration verification
2. **SON:** Integration with SONCAP portal (if available)
3. **Nigeria Customs Service:** Portal access for SGD submission
4. **Akwa Ibom State Government:** Official endorsement and support
5. **Local Banks:** For duty payment integration
6. **Logistics Partners:** State-level shipping and clearing agents

This comprehensive plan provides a full roadmap for deploying the Akwa Ibom State AfriXport platform with integrated NEPC, SON, and Customs compliance. The implementation leverages the existing robust compliance infrastructure (USA, EU, UK) and extends it with Nigerian-specific regulatory requirements.

The system would create a seamless experience for vendors in Akwa Ibom State to manage all their export compliance requirements in one place, from NEPC registration to SON product certification to Nigeria Customs documentation.

Would you like me to proceed with implementing any specific phase of this plan?
