# EcoCaptain V0 - Testing & QA Strategy

## Overview

This document outlines the comprehensive testing and quality assurance strategy for EcoCaptain V0. Our goal is to achieve 95% code coverage on critical paths while maintaining high quality standards and ensuring the platform is secure, performant, and accessible.

## Testing Philosophy

### Core Principles
1. **Shift-Left Testing**: Test early and often in the development cycle
2. **Test Pyramid**: More unit tests, fewer E2E tests
3. **Continuous Testing**: Automated tests run on every commit
4. **Risk-Based Testing**: Focus on critical user journeys
5. **Accessibility First**: WCAG 2.1 AA compliance from the start

## Test Coverage Requirements

### Coverage Targets
- **Overall**: 85% minimum
- **Critical Paths**: 95% minimum
- **New Code**: 90% minimum
- **Backend Services**: 90% minimum
- **Frontend Components**: 85% minimum
- **API Endpoints**: 100% required

### Critical Paths (95% coverage required)
1. Authentication and authorization flows
2. Impact claim creation and calculation
3. Evidence upload and verification
4. Report generation
5. Payment processing
6. Data export/import

## Testing Pyramid

```
          /\
         /  \        5% - Exploratory Testing
        /    \
       /------\      10% - End-to-End Tests
      /        \
     /----------\    25% - Integration Tests
    /            \
   /--------------\  60% - Unit Tests
  /________________\
```

## Test Types and Tools

### Unit Testing

**Coverage Target**: 60% of all tests

#### Backend (Node.js)
```javascript
// Example: Impact calculation unit test
describe('ImpactCalculator', () => {
  describe('calculateSROI', () => {
    it('should calculate correct SROI ratio', () => {
      const inputs = {
        socialValue: 100000,
        investment: 25000,
        discountRate: 0.035,
        timeHorizon: 5
      };
      
      const result = calculator.calculateSROI(inputs);
      
      expect(result.ratio).toBeCloseTo(3.86, 2);
      expect(result.netPresentValue).toBeCloseTo(75000, 0);
    });
    
    it('should handle edge cases', () => {
      expect(() => calculator.calculateSROI({ investment: 0 }))
        .toThrow('Investment cannot be zero');
    });
  });
});
```

#### Frontend (React)
```javascript
// Example: Component unit test
import { render, screen, fireEvent } from '@testing-library/react';
import ImpactClaimForm from './ImpactClaimForm';

describe('ImpactClaimForm', () => {
  it('should validate required fields', async () => {
    render(<ImpactClaimForm />);
    
    const submitButton = screen.getByRole('button', { name: /submit/i });
    fireEvent.click(submitButton);
    
    expect(await screen.findByText(/indicator is required/i)).toBeInTheDocument();
    expect(await screen.findByText(/value is required/i)).toBeInTheDocument();
  });
  
  it('should submit valid form data', async () => {
    const onSubmit = jest.fn();
    render(<ImpactClaimForm onSubmit={onSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/indicator/i), {
      target: { value: 'CO2 Reduction' }
    });
    fireEvent.change(screen.getByLabelText(/value/i), {
      target: { value: '1000' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        indicator: 'CO2 Reduction',
        value: 1000
      });
    });
  });
});
```

### Integration Testing

**Coverage Target**: 25% of all tests

#### API Integration Tests
```javascript
// Example: GraphQL integration test
describe('GraphQL API', () => {
  let server;
  
  beforeAll(async () => {
    server = await createTestServer();
  });
  
  afterAll(async () => {
    await server.close();
  });
  
  it('should create impact claim with evidence', async () => {
    const CREATE_CLAIM = gql`
      mutation CreateClaim($input: CreateClaimInput!) {
        createImpactClaim(input: $input) {
          id
          indicator
          value
          assuranceLevel
        }
      }
    `;
    
    const result = await server.executeOperation({
      query: CREATE_CLAIM,
      variables: {
        input: {
          projectId: 'test-project-id',
          indicator: 'Water Saved',
          value: 50000,
          unit: 'liters',
          method: 'SROI'
        }
      }
    });
    
    expect(result.errors).toBeUndefined();
    expect(result.data.createImpactClaim).toMatchObject({
      indicator: 'Water Saved',
      value: 50000,
      assuranceLevel: 'self'
    });
  });
});
```

### End-to-End Testing

**Coverage Target**: 10% of all tests

#### Cypress E2E Tests
```javascript
// Example: Critical user journey
describe('Impact Claim Creation Flow', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password');
    cy.visit('/projects/test-project');
  });
  
  it('should complete full claim creation with verification', () => {
    // Navigate to claims
    cy.get('[data-cy=nav-claims]').click();
    cy.get('[data-cy=create-claim-btn]').click();
    
    // Fill claim form
    cy.get('[data-cy=indicator-select]').select('Carbon Reduction');
    cy.get('[data-cy=value-input]').type('1500');
    cy.get('[data-cy=unit-select]').select('tCO2e');
    cy.get('[data-cy=method-select]').select('SROI');
    
    // Add calculation details
    cy.get('[data-cy=calculation-tab]').click();
    cy.get('[data-cy=social-value]').type('75000');
    cy.get('[data-cy=investment]').type('20000');
    
    // Upload evidence
    cy.get('[data-cy=evidence-tab]').click();
    cy.get('[data-cy=upload-evidence]').attachFile('evidence.pdf');
    
    // Submit and verify
    cy.get('[data-cy=submit-claim]').click();
    cy.get('[data-cy=success-message]').should('be.visible');
    
    // Verify in list
    cy.get('[data-cy=claims-list]').should('contain', 'Carbon Reduction');
    cy.get('[data-cy=claims-list]').should('contain', '1,500 tCO2e');
  });
});
```

### Performance Testing

#### Load Testing with K6
```javascript
// Example: API load test
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '5m', target: 100 },  // Ramp up
    { duration: '10m', target: 100 }, // Stay at 100 users
    { duration: '5m', target: 200 },  // Spike to 200
    { duration: '10m', target: 200 }, // Stay at 200
    { duration: '5m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.1'],    // Error rate under 10%
  },
};

export default function() {
  const payload = JSON.stringify({
    query: `query { projects { id name impactClaims { id value } } }`
  });
  
  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${__ENV.API_TOKEN}`,
    },
  };
  
  const response = http.post('https://api.ecocaptain.com/graphql', payload, params);
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'has projects data': (r) => JSON.parse(r.body).data.projects !== undefined,
  });
  
  sleep(1);
}
```

### Security Testing

#### OWASP ZAP Security Scan
```yaml
# .github/workflows/security.yml
name: Security Testing

on:
  schedule:
    - cron: '0 2 * * *' # Daily at 2 AM

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    steps:
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'https://staging.ecocaptain.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a -j -l WARN'
          
      - name: Upload ZAP Report
        uses: actions/upload-artifact@v2
        with:
          name: zap-report
          path: report_html.html
```

## CI/CD Testing Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:unit
      - uses: codecov/codecov-action@v2
        with:
          flags: unit
          
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm run db:migrate:test
      - run: npm run test:integration
      - uses: codecov/codecov-action@v2
        with:
          flags: integration
          
  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm run build
      - run: npm run start:test &
      - run: npx wait-on http://localhost:3000
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v2
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots
          
  accessibility-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm run test:a11y
      - run: npm run lighthouse:ci
      
  security-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm audit --audit-level=high
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
```

## Test Reporting

### Coverage Reports
```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.{js,ts}',
    '!src/**/*.stories.{js,jsx,ts,tsx}'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 85,
      statements: 85
    },
    './src/services/impact/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95
    }
  },
  coverageReporters: ['text', 'lcov', 'html', 'json']
};
```

## Bug Management

### Bug Severity Levels

| Level | Description | Response Time | Example |
|-------|-------------|---------------|----------|
| P0 - Critical | System down, data loss | 2 hours | Authentication broken |
| P1 - High | Major feature broken | 4 hours | Cannot create claims |
| P2 - Medium | Feature partially broken | 1 day | Report formatting issue |
| P3 - Low | Minor issue | 1 week | UI alignment problem |

## Quality Gates

### Definition of Done
- [ ] Code complete and follows standards
- [ ] Unit tests written and passing (>90% coverage)
- [ ] Integration tests passing
- [ ] Code reviewed by 2+ developers
- [ ] No critical security vulnerabilities
- [ ] Documentation updated
- [ ] Accessibility tested (WCAG 2.1 AA)
- [ ] Performance benchmarks met
- [ ] Deployed to staging environment
- [ ] Product owner acceptance

### Release Criteria
- All P0 and P1 bugs resolved
- 95% test coverage on critical paths
- Performance tests passing
- Security scan clean
- Documentation complete
- Rollback plan in place

## Conclusion

This testing and QA strategy ensures EcoCaptain V0 meets the highest quality standards while maintaining development velocity. Regular reviews and continuous improvement will keep our testing practices aligned with project needs and industry best practices.