# EcoCaptain V0

## Environmental Impact Tracking Platform

EcoCaptain is a comprehensive environmental impact tracking platform with dual-rail architecture supporting both community-led and institutional users. The platform features an Evidence Graph for claims tracking, progressive assurance ladder, and compliance-ready reporting.

## 🎯 Vision

To create a unified platform that bridges community-driven environmental action with institutional compliance requirements, enabling transparent, verifiable, and impactful sustainability tracking.

## 🚀 Key Features

### Dual-Rail Architecture
- **Community Rail**: Mobile/offline capture, local dashboards, participatory workflows (PIPA)
- **Institutional Rail**: Enterprise reporting, AASB/ISSB compliance, third-party verification

### Evidence Graph & Assurance
- Visual representation of impact claims and evidence chains
- Three-tier assurance ladder (self-declared → peer-reviewed → third-party verified)
- Blockchain integration for immutable records

### Impact Measurement Methods
- **SROI** (Social Return on Investment) calculator
- **CBA** (Cost-Benefit Analysis) aligned with Treasury guidelines
- **PIPA** (Participatory Impact Pathways Analysis) toolkit
- **SDG** tagging and contribution tracking

### Compliance & Reporting
- AASB S2 / ISSB aligned reporting templates
- TNFD-ready nature disclosures
- SD VISta certification preparation
- Automated ESG reporting dashboards

## 🏗️ Technology Stack

### Frontend
- **React 18** with functional components and hooks
- **TypeScript** for type safety
- **Material-UI** component library
- **D3.js** for Evidence Graph visualization
- **Redux Toolkit** for state management

### Backend
- **Node.js** with Express
- **GraphQL** with Apollo Server
- **PostgreSQL** with PostGIS for geospatial data
- **Redis** for caching and sessions
- **Bull** for job queues

### Infrastructure
- **Docker** containers
- **Kubernetes** orchestration
- **AWS** cloud services
- **GitHub Actions** CI/CD

## 📊 Project Status

### Current Phase: Foundation & Architecture
- Setting up core infrastructure
- Implementing authentication system
- Building database schema
- Creating API foundation

### Milestones
1. ✅ **Phase 1**: Foundation & Architecture (Weeks 1-4)
2. ⏳ **Phase 2**: Core Platform Development (Weeks 5-10)
3. 📅 **Phase 3**: Evidence Graph & Assurance (Weeks 11-16)
4. 📅 **Phase 4**: Impact Methods & Compliance (Weeks 17-20)
5. 📅 **Phase 5**: Testing & Launch (Weeks 21-24)

## 🚦 Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Redis 6+
- Docker & Docker Compose

### Installation

```bash
# Clone the repository
git clone https://github.com/petemoulton/EcoCaptain-V0.git
cd EcoCaptain-V0

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
```

### Development

```bash
# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Run linting
npm run lint

# Run type checking
npm run type-check

# Build for production
npm run build
```

## 📖 Documentation

- [Strategic Plan](./docs/STRATEGIC_PLAN.md)
- [Technical Architecture](./docs/TECHNICAL_ARCHITECTURE.md)
- [Testing & QA Strategy](./docs/TESTING_QA_STRATEGY.md)
- [API Documentation](./docs/API.md)
- [Contributing Guide](./CONTRIBUTING.md)

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details on:
- Code of Conduct
- Development workflow
- Pull request process
- Coding standards

## 📊 Project Metrics

![Test Coverage](https://img.shields.io/badge/coverage-0%25-red)
![Build Status](https://img.shields.io/badge/build-pending-yellow)
![License](https://img.shields.io/badge/license-MIT-green)

## 🔗 Links

- [Project Board](https://github.com/petemoulton/EcoCaptain-V0/projects)
- [Issues](https://github.com/petemoulton/EcoCaptain-V0/issues)
- [Wiki](https://github.com/petemoulton/EcoCaptain-V0/wiki)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

## 🙏 Acknowledgments

- Australian Government Treasury for CBA guidelines
- Social Value International for SROI methodology
- Gold Standard for SDG Impact Tools
- Verra for SD VISta framework

---

**EcoCaptain V0** - Building transparent, verifiable environmental impact tracking for Australia and beyond.