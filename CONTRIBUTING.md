# Contributing to Bang For Your Buck

Thank you for considering contributing to Bang For Your Buck! We're building Ireland's most comprehensive price tracking platform, and we need your help to make it great.

## 🌟 Ways to Contribute

- 🐛 **Report bugs** - Found something broken? Let us know
- 💡 **Suggest features** - Have an idea? We want to hear it
- 📝 **Improve documentation** - Help make our docs clearer
- 🎨 **Design improvements** - Better UX/UI suggestions
- 💻 **Write code** - Fix bugs or implement features
- 🧪 **Write tests** - Help us maintain quality
- 📊 **Submit data** - Add venues, submit prices
- 🌍 **Translate** - Help us reach more people (future)

## 🚀 Getting Started

### First Time Contributors

Welcome! Here's how to get started:

1. **Read the README** - Understand the project
2. **Set up your environment** - Follow the [Development Setup](README.md#development-setup)
3. **Find an issue** - Look for `good first issue` labels
4. **Ask questions** - Use GitHub Discussions or Issues

### Finding Work

Check these labels on our [Issues page](https://github.com/kimatron/bangforyourbuck/issues):
- `good first issue` - Perfect for newcomers
- `help wanted` - We need assistance on these
- `bug` - Something isn't working
- `enhancement` - New features or improvements
- `documentation` - Documentation improvements

## 📋 Before You Start

1. **Check existing issues** - Someone might already be working on it
2. **Comment on the issue** - Let us know you're interested
3. **Wait for assignment** - We'll assign it to you to avoid duplicates
4. **Fork the repo** - Create your own copy

## 🔧 Development Process

### 1. Fork and Clone
```bash
# Fork the repo on GitHub, then:
git clone https://github.com/YOUR_USERNAME/bangforyourbuck.git
cd bangforyourbuck

# Add upstream remote
git remote add upstream https://github.com/ORIGINAL_OWNER/bangforyourbuck.git
```

### 2. Create a Branch
```bash
# Update your main branch
git checkout main
git pull upstream main

# Create a feature branch
git checkout -b feature/your-feature-name
# or
git checkout -b fix/bug-description
```

**Branch Naming Convention:**
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Adding/updating tests
- `chore/` - Maintenance tasks

Examples:
- `feature/add-coffee-category`
- `fix/map-marker-clustering`
- `docs/update-api-examples`

### 3. Make Your Changes

#### Code Style

**Python (Backend):**
```bash
# Format code
black .

# Sort imports
isort .

# Lint
flake8 .

# Type checking (optional)
mypy .
```

**JavaScript/TypeScript (Frontend):**
```bash
# Format code
npm run format

# Lint
npm run lint

# Type checking
npm run type-check
```

#### Writing Good Code

- **Keep it simple** - Write code that's easy to understand
- **Comment complex logic** - Explain why, not what
- **Follow existing patterns** - Be consistent with the codebase
- **Write tests** - All new features need tests
- **Keep commits focused** - One logical change per commit

#### Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation only
- `style` - Code style (formatting, no logic change)
- `refactor` - Code restructuring
- `perf` - Performance improvements
- `test` - Adding/updating tests
- `chore` - Maintenance (dependencies, config)
- `ci` - CI/CD changes

**Examples:**
```bash
feat(venues): add search autocomplete

Implemented debounced autocomplete search for venue names.
Uses PostgreSQL full-text search for better performance.

Closes #123

fix(api): handle missing GPS coordinates gracefully

Previously crashed when latitude/longitude were null.
Now falls back to venue address coordinates.

Fixes #456

docs(readme): update deployment instructions

Added Railway deployment steps and environment variable guide.
```

**Commit Message Rules:**
- Use imperative mood ("add" not "added" or "adds")
- First line max 72 characters
- Blank line between subject and body
- Reference issues in footer

### 4. Write Tests

All new code must include tests.

**Backend Tests:**
```python
# apps/venues/tests/test_views.py
from django.test import TestCase
from rest_framework.test import APIClient

class VenueAPITestCase(TestCase):
    def setUp(self):
        self.client = APIClient()
        
    def test_list_venues(self):
        """Test listing venues returns paginated results"""
        response = self.client.get('/api/venues/')
        self.assertEqual(response.status_code, 200)
        self.assertIn('results', response.json())
        
    def test_search_venues(self):
        """Test venue search by name"""
        response = self.client.get('/api/venues/?search=temple')
        self.assertEqual(response.status_code, 200)
        results = response.json()['results']
        self.assertTrue(all('temple' in v['name'].lower() for v in results))
```

**Frontend Tests:**
```typescript
// src/components/VenueCard.test.tsx
import { render, screen } from '@testing-library/react';
import { VenueCard } from './VenueCard';

describe('VenueCard', () => {
  it('renders venue information correctly', () => {
    const venue = {
      id: '123',
      name: 'The Temple Bar',
      city: 'Dublin',
      average_price: 6.50,
    };
    
    render(<VenueCard venue={venue} />);
    
    expect(screen.getByText('The Temple Bar')).toBeInTheDocument();
    expect(screen.getByText('Dublin')).toBeInTheDocument();
    expect(screen.getByText('€6.50')).toBeInTheDocument();
  });
});
```

### 5. Run Tests
```bash
# Backend
cd backend
python manage.py test
coverage run --source='.' manage.py test
coverage report

# Frontend
cd frontend
npm test
npm test -- --coverage
```

**Test Coverage Requirements:**
- New code: >80% coverage
- Critical paths: 100% coverage
- All tests must pass before PR

### 6. Update Documentation

If your changes affect:
- **API endpoints** - Update `docs/API.md`
- **User features** - Update `README.md`
- **Setup process** - Update relevant docs
- **Configuration** - Update env examples

### 7. Push Changes
```bash
# Commit your changes
git add .
git commit -m "feat(venues): add search autocomplete"

# Push to your fork
git push origin feature/your-feature-name
```

### 8. Create Pull Request

1. Go to your fork on GitHub
2. Click "New Pull Request"
3. Select your branch
4. Fill out the PR template (see below)
5. Click "Create Pull Request"

## 📝 Pull Request Guidelines

### PR Template
```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to change)
- [ ] Documentation update

## Related Issue
Closes #(issue number)

## How Has This Been Tested?
Describe the tests you ran.

## Screenshots (if applicable)
Add screenshots for UI changes.

## Checklist
- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
- [ ] Any dependent changes have been merged and published
```

### PR Best Practices

**Do:**
- ✅ Keep PRs focused (one feature/fix per PR)
- ✅ Write clear descriptions
- ✅ Reference related issues
- ✅ Add screenshots for UI changes
- ✅ Update tests and documentation
- ✅ Respond to review feedback promptly
- ✅ Keep commits clean and meaningful

**Don't:**
- ❌ Mix multiple unrelated changes
- ❌ Submit untested code
- ❌ Ignore CI failures
- ❌ Force push after review started
- ❌ Leave unresolved merge conflicts

### Review Process

1. **Automated Checks**
   - Tests must pass
   - Linting must pass
   - Coverage must meet requirements

2. **Code Review**
   - At least one maintainer approval required
   - Address all review comments
   - Re-request review after changes

3. **Merge**
   - Maintainer will merge when approved
   - Squash and merge for cleaner history
   - Your contribution is now live! 🎉

## 🐛 Reporting Bugs

### Before Reporting

1. **Search existing issues** - Bug might already be reported
2. **Try latest version** - Bug might already be fixed
3. **Reproduce the bug** - Make sure it's consistent

### Bug Report Template
```markdown
**Describe the bug**
A clear description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '...'
3. Scroll down to '...'
4. See error

**Expected behavior**
What you expected to happen.

**Screenshots**
Add screenshots if applicable.

**Environment:**
 - OS: [e.g., macOS 14.0]
 - Browser: [e.g., Chrome 119]
 - Version: [e.g., 1.0.0]

**Additional context**
Any other relevant information.
```

## 💡 Suggesting Features

### Before Suggesting

1. **Check existing suggestions** - Someone might have suggested it
2. **Check roadmap** - It might already be planned
3. **Consider scope** - Is it aligned with project goals?

### Feature Request Template
```markdown
**Is your feature request related to a problem?**
A clear description of the problem.

**Describe the solution you'd like**
What you want to happen.

**Describe alternatives you've considered**
Other solutions you've thought about.

**Additional context**
Mockups, examples, or other relevant information.

**Why is this valuable?**
Who would benefit and how?
```

## 📊 Submitting Data

Help us grow the database!

### Adding Venues

1. Check venue doesn't already exist
2. Gather accurate information:
   - Full name
   - Complete address
   - Phone and website (if available)
   - Venue type (pub, hotel bar, etc.)
3. Submit via Django admin (if you have access) or create an issue

### Submitting Prices

1. Use the web app submission form
2. Be accurate - verify the price
3. Add photo of receipt when possible
4. Include context in comments if helpful

### Data Quality Guidelines

- **Accuracy** - Double-check all information
- **Completeness** - Fill in all available fields
- **Current** - Only submit recent prices
- **Honest** - Never submit fake data

## 🎨 Design Contributions

We welcome design improvements!

### What We Need

- UI/UX improvements
- Visual design enhancements
- Accessibility improvements
- Mobile experience optimization
- Branding and identity

### How to Contribute

1. Create mockups/wireframes (Figma preferred)
2. Open an issue with your designs
3. Discuss with maintainers
4. If approved, implement or help implement

## 🌍 Translation (Future)

We plan to support Irish (Gaeilge) and other languages.

### Translation Guidelines

- Maintain tone and meaning
- Consider cultural context
- Use appropriate formality level
- Keep technical terms consistent

## 📜 Code of Conduct

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behavior includes:**
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate in a professional setting

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting the project team at conduct@bangforyourbuck.ie. All complaints will be reviewed and investigated promptly and fairly.

Project maintainers have the right and responsibility to remove, edit, or reject comments, commits, code, wiki edits, issues, and other contributions that are not aligned with this Code of Conduct.

## 🏆 Recognition

We value all contributions! Contributors will be:
- Listed in `CONTRIBUTORS.md`
- Mentioned in release notes (for significant contributions)
- Eligible for contributor badges (Phase 2)
- Featured on our website (with permission)

## 💬 Communication Channels

- **GitHub Issues** - Bug reports, feature requests
- **GitHub Discussions** - General questions, ideas, community chat
- **Email** - hello@bangforyourbuck.ie
- **Twitter** - [@bangforbuckIE](https://twitter.com/bangforbuckIE)

### Response Times

We aim to respond to:
- Critical bugs: Within 24 hours
- Pull requests: Within 3 days
- Issues: Within 5 days
- Questions: Within 7 days

*Note: We're a small team/community project, so please be patient!*

## 🎓 Learning Resources

New to contributing to open source? Check these out:

### Git & GitHub
- [GitHub's Git Guide](https://github.com/git-guides)
- [First Contributions](https://github.com/firstcontributions/first-contributions)
- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)

### Django
- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework Tutorial](https://www.django-rest-framework.org/tutorial/quickstart/)
- [Real Python Django Tutorials](https://realpython.com/tutorials/django/)

### React
- [React Documentation](https://react.dev/)
- [React + TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)

## 🔍 Code Review Checklist

### For Reviewers

When reviewing PRs, check for:

**Functionality:**
- [ ] Code works as described
- [ ] Edge cases handled
- [ ] No breaking changes (or documented)
- [ ] Error handling is appropriate

**Code Quality:**
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Complex logic is commented
- [ ] Variable/function names are clear
- [ ] No unnecessary code

**Testing:**
- [ ] Tests exist and pass
- [ ] Edge cases are tested
- [ ] Coverage is adequate
- [ ] Tests are maintainable

**Documentation:**
- [ ] README updated if needed
- [ ] API docs updated if needed
- [ ] Comments explain "why" not "what"
- [ ] Docstrings for public functions

**Security:**
- [ ] No sensitive data exposed
- [ ] Input validation present
- [ ] SQL injection prevented
- [ ] XSS vulnerabilities prevented

**Performance:**
- [ ] No obvious performance issues
- [ ] Database queries optimized
- [ ] Large operations are async
- [ ] Caching used appropriately

### For Contributors

Before requesting review:

- [ ] I've tested my changes locally
- [ ] All tests pass
- [ ] Linting passes
- [ ] I've updated documentation
- [ ] I've added tests for new features
- [ ] I've rebased on latest main
- [ ] Commit messages follow convention
- [ ] PR description is complete
- [ ] No merge conflicts

## 📦 Release Process

### Version Numbers

We use [Semantic Versioning](https://semver.org/):
- **MAJOR.MINOR.PATCH** (e.g., 1.2.3)
- **MAJOR** - Breaking changes
- **MINOR** - New features (backward compatible)
- **PATCH** - Bug fixes (backward compatible)

### Release Schedule

- **Patch releases** - As needed (bug fixes)
- **Minor releases** - Monthly (new features)
- **Major releases** - When breaking changes necessary

### Contributing to Releases

Contributors cannot directly create releases, but you can:
- Tag issues with milestone labels
- Help with release notes
- Test release candidates

## 🚫 What We Don't Accept

To keep the project focused, we generally don't accept:

- **Off-topic features** - Must align with project goals
- **Low-quality code** - Untested, poorly written, or insecure
- **Incomplete work** - Half-finished features without plan
- **Breaking changes** - Without very strong justification
- **Plagiarized code** - Must be your own work or properly attributed
- **Dependencies with restrictive licenses** - Must be MIT-compatible

If in doubt, **open an issue first** to discuss!

## 🎯 Priority Areas

We especially need help with:

**High Priority:**
- [ ] Venue data collection (Dublin, Cork, Galway)
- [ ] Mobile responsiveness improvements
- [ ] Test coverage improvements
- [ ] Performance optimization
- [ ] Accessibility (WCAG 2.1 AA compliance)

**Medium Priority:**
- [ ] API documentation improvements
- [ ] Admin panel UX improvements
- [ ] Additional chart types for statistics
- [ ] Email notification templates
- [ ] SEO optimization

**Future/Low Priority:**
- [ ] Native mobile apps
- [ ] Internationalization (i18n)
- [ ] Advanced analytics
- [ ] Machine learning features
- [ ] Public API for third parties

## 🐛 Common Issues & Solutions

### Backend Issues

**Problem:** Database migration errors
```bash
# Solution: Reset database (development only!)
python manage.py flush
python manage.py migrate
python manage.py loaddata fixtures/seed_data.json
```

**Problem:** PostGIS extension not found
```bash
# Solution: Install PostGIS
# macOS:
brew install postgis
# Ubuntu:
sudo apt-get install postgis

# Then in psql:
CREATE EXTENSION postgis;
```

**Problem:** Celery tasks not running
```bash
# Solution: Ensure Redis is running
redis-cli ping  # Should return PONG

# Start Celery worker
celery -A config worker -l info
```

### Frontend Issues

**Problem:** Module not found errors
```bash
# Solution: Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

**Problem:** Map not displaying
```bash
# Solution: Check API key in .env.local
echo $VITE_GOOGLE_MAPS_API_KEY

# Ensure key has correct API enabled on Google Cloud Console
```

**Problem:** TypeScript errors
```bash
# Solution: Rebuild types
npm run type-check
# If errors persist, restart TypeScript server in VS Code
```

### Git Issues

**Problem:** Merge conflicts
```bash
# Solution: Resolve conflicts manually
git status  # See conflicted files
# Edit files, resolve conflicts
git add .
git commit -m "chore: resolve merge conflicts"
```

**Problem:** Accidentally committed to main
```bash
# Solution: Move commits to new branch
git branch feature/my-feature
git reset --hard origin/main
git checkout feature/my-feature
```

**Problem:** Need to update PR with latest main
```bash
# Solution: Rebase on main
git checkout main
git pull upstream main
git checkout your-branch
git rebase main
# Resolve any conflicts
git push --force-with-lease origin your-branch
```

## 📚 Additional Documentation

- [Architecture Overview](docs/ARCHITECTURE.md) *(coming soon)*
- [Database Schema](docs/DATABASE.md) *(coming soon)*
- [API Documentation](docs/API.md) ✅
- [Deployment Guide](docs/DEPLOYMENT.md) ✅
- [Security Policy](SECURITY.md) *(coming soon)*

## ❓ FAQ

### Can I contribute without coding?

**Yes!** Non-code contributions are valuable:
- Submit venue data and prices
- Report bugs you encounter
- Suggest features and improvements
- Improve documentation
- Share the project on social media
- Help other users in Discussions

### How long until my PR is reviewed?

We aim for 3 days, but it depends on:
- PR complexity
- Current maintainer workload
- How well PR follows guidelines

To speed up review:
- Keep PRs small and focused
- Write clear descriptions
- Ensure all checks pass
- Respond quickly to feedback

### Can I work on multiple issues?

Yes, but:
- Finish one before starting another
- Keep PRs separate (one per issue)
- Communicate if you need more time

### My PR was closed, why?

Common reasons:
- Doesn't align with project goals
- Incomplete or poor quality
- No response to review feedback
- Duplicate of existing work
- Better solution found

Don't be discouraged! Learn from feedback and try again.

### How do I become a maintainer?

Maintainers are invited based on:
- Consistent, high-quality contributions
- Understanding of project goals
- Helpful in community discussions
- Trusted by existing team

There's no formal application - just keep contributing!

### Can I use this code in my own project?

Yes! The project is MIT licensed. You can:
- Use it commercially
- Modify it
- Distribute it
- Sublicense it

Just include the original license and copyright.

### I have a question not covered here

Ask in:
- [GitHub Discussions](https://github.com/kimatron/bangforyourbuck/discussions) - For general questions
- [GitHub Issues](https://github.com/kimatron/bangforyourbuck/issues) - For bug reports or features
- Email: hello@bangforyourbuck.ie - For private inquiries

## 🙏 Thank You!

Every contribution, no matter how small, makes a difference. Whether you're fixing a typo, reporting a bug, or building a major feature - **thank you** for helping make Bang For Your Buck better for everyone!

Together, we're bringing price transparency to Ireland! 🇮🇪

---

**Ready to contribute?** Pick an issue and get started! If you're unsure where to begin, look for the `good first issue` label.

**Questions?** Don't hesitate to ask in Discussions or open an issue.

**Happy coding!** 🚀
