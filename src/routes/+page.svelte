<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import {
		Shield,
		Users,
		AlertTriangle,
		Ban,
		Activity,
		Clock,
		MessageSquare,
		XCircle,
		Loader,
		ChevronLeft,
		ChevronRight
	} from 'lucide-svelte';

	const API_BASE = 'https://telegram-group-moderator-bot.onrender.com';

	interface Stats {
		totalUsers: number;
		totalWarnings: number;
		totalViolations: number;
		totalBans: number;
		totalWarningsIssued: number;
	}

	interface UserData {
		id: number;
		telegramId: string;
		username: string;
		firstName: string;
		warningCount: number;
	}

	interface Violation {
		id: number;
		userId: number;
		message: string;
		violationType: string;
		actionTaken: string;
		createdAt: string;
		user: {
			telegramId: string;
			username: string;
			firstName: string;
		};
	}

	let stats: Stats = $state({
		totalUsers: 0,
		totalWarnings: 0,
		totalViolations: 0,
		totalBans: 0,
		totalWarningsIssued: 0
	});

	let users: UserData[] = $state([]);
	let violations: Violation[] = $state([]);
	let realtimeEvents: any[] = $state([]);
	let eventSource: EventSource | null = null;
	let loading = $state(true);
	let connected = $state(false);

	// Pagination state
	let violationsPage = $state(1);
	let usersPage = $state(1);
	const violationsPerPage = 10;
	const usersPerPage = 10;

	async function fetchStats() {
		try {
			const response = await fetch(`${API_BASE}/api/stats`);
			const data = await response.json();
			if (data.success) {
				stats = data.data;
			}
		} catch (error) {
			console.error('Error fetching stats:', error);
		}
	}

	async function fetchUsers() {
		try {
			const response = await fetch(`${API_BASE}/api/users`);
			const data = await response.json();
			if (data.success) {
				users = data.data.slice(0, 10);
			}
		} catch (error) {
			console.error('Error fetching users:', error);
		}
	}

	async function fetchViolations() {
		try {
			const response = await fetch(`${API_BASE}/api/logs?limit=15`);
			const data = await response.json();
			if (data.success) {
				violations = data.data;
			}
		} catch (error) {
			console.error('Error fetching violations:', error);
		}
	}

	function connectToEventStream() {
		eventSource = new EventSource(`${API_BASE}/api/events`);

		eventSource.onmessage = (event) => {
			const data = JSON.parse(event.data);
			if (data.type === 'connected') {
				connected = true;
			} else {
				realtimeEvents = [data, ...realtimeEvents].slice(0, 5);
				fetchStats();
				fetchUsers();
				fetchViolations();
			}
		};

		eventSource.onerror = () => {
			connected = false;
			setTimeout(() => {
				if (eventSource) {
					eventSource.close();
					connectToEventStream();
				}
			}, 5000);
		};
	}

	onMount(async () => {
		await Promise.all([fetchStats(), fetchUsers(), fetchViolations()]);
		loading = false;
		connectToEventStream();
	});

	onDestroy(() => {
		if (eventSource) {
			eventSource.close();
		}
	});

	function formatDate(dateString: string) {
		const date = new Date(dateString);
		const now = new Date();
		const diff = now.getTime() - date.getTime();
		const seconds = Math.floor(diff / 1000);
		const minutes = Math.floor(seconds / 60);
		const hours = Math.floor(minutes / 60);
		const days = Math.floor(hours / 24);

		if (days > 0) return `${days}d ago`;
		if (hours > 0) return `${hours}h ago`;
		if (minutes > 0) return `${minutes}m ago`;
		return 'Just now';
	}

	function getAvatarUrl(username: string) {
		return `https://api.dicebear.com/7.x/avataaars/svg?seed=${username}`;
	}

	// Pagination computed values
	let paginatedViolations = $derived(
		violations.slice(
			(violationsPage - 1) * violationsPerPage,
			violationsPage * violationsPerPage
		)
	);

	let paginatedUsers = $derived(
		users.slice((usersPage - 1) * usersPerPage, usersPage * usersPerPage)
	);

	let totalViolationsPages = $derived(Math.ceil(violations.length / violationsPerPage));
	let totalUsersPages = $derived(Math.ceil(users.length / usersPerPage));
</script>

<svelte:head>
	<title>Group Guard - Telegram Moderation Bot Dashboard</title>
	<meta
		name="description"
		content="Real-time dashboard for Group Guard - AI-powered Telegram group moderation bot"
	/>
</svelte:head>

{#if loading}
	<div class="loading-container">
		<div class="loading-spinner">
			<Loader size={48} />
		</div>
		<p>Loading dashboard...</p>
	</div>
{:else}
	<!-- Hero Section -->
	<section class="hero">
		<div class="container">
			<div class="hero-content">
				<div class="hero-badge">
					<div class="status-dot {connected ? 'connected' : ''}"></div>
					<span>{connected ? 'Live' : 'Connecting...'}</span>
				</div>

				<h1 class="hero-title">
					<img src="/bot.jpg" alt="Group Guard" class="hero-icon" />
					Group Guard
				</h1>

				<p class="hero-description">
					AI-powered Telegram moderation bot that automatically detects and removes inappropriate
					content, keeps your community safe, and maintains a healthy group environment.
				</p>

				<a href="https://kentom.co.ke" target="_blank" rel="noopener" class="kentom-link">
					<img src="https://kentom.co.ke/logo-light.png" alt="Kentom" class="kentom-logo" />
					<span>Created by Kentom</span>
				</a>
			</div>
		</div>
	</section>

	<!-- Stats Section -->
	<section class="stats-section">
		<div class="container">
			<h2 class="section-title">Moderation Statistics</h2>

			<div class="stats-grid">
				<div class="stat-card">
					<div class="stat-icon users">
						<Users size={24} />
					</div>
					<div class="stat-content">
						<span class="stat-label">Total Users</span>
						<span class="stat-value">{stats.totalUsers.toLocaleString()}</span>
					</div>
				</div>

				<div class="stat-card">
					<div class="stat-icon warnings">
						<AlertTriangle size={24} />
					</div>
					<div class="stat-content">
						<span class="stat-label">Warnings Issued</span>
						<span class="stat-value">{stats.totalWarnings.toLocaleString()}</span>
					</div>
				</div>

				<div class="stat-card">
					<div class="stat-icon violations">
						<Shield size={24} />
					</div>
					<div class="stat-content">
						<span class="stat-label">Violations Detected</span>
						<span class="stat-value">{stats.totalViolations.toLocaleString()}</span>
					</div>
				</div>

				<div class="stat-card">
					<div class="stat-icon bans">
						<Ban size={24} />
					</div>
					<div class="stat-content">
						<span class="stat-label">Users Banned</span>
						<span class="stat-value">{stats.totalBans.toLocaleString()}</span>
					</div>
				</div>
			</div>
		</div>
	</section>

	<!-- Real-time Activity Banner -->
	{#if realtimeEvents.length > 0}
		<div class="activity-banner">
			<div class="container">
				<div class="activity-content">
					<div class="activity-icon">
						<Activity size={20} />
					</div>
					<div class="activity-info">
						<strong>Live Activity:</strong>
						{#if realtimeEvents[0].type === 'warning'}
							User {realtimeEvents[0].data.user.username
								? `@${realtimeEvents[0].data.user.username}`
								: realtimeEvents[0].data.user.firstName || 'Unknown'} received a warning
						{:else if realtimeEvents[0].type === 'ban'}
							User {realtimeEvents[0].data.user.username
								? `@${realtimeEvents[0].data.user.username}`
								: realtimeEvents[0].data.user.firstName || 'Unknown'} was banned
						{:else}
							New violation detected
						{/if}
					</div>
					<div class="activity-time">Just now</div>
				</div>
			</div>
		</div>
	{/if}

	<!-- Main Content -->
	<section class="content-section">
		<div class="container">
			<div class="content-grid">
				<!-- Violations -->
				<div class="violations-panel">
					<div class="panel-header">
						<div class="panel-title">
							<MessageSquare size={20} />
							<h3>Recent Violations</h3>
						</div>
						<div class="panel-badge">{violations.length} total</div>
					</div>

					<div class="violations-list">
						{#each paginatedViolations as violation (violation.id)}
							<div class="violation-card">
								<div class="violation-header">
									<img
										src={getAvatarUrl(violation.user.username)}
										alt={violation.user.firstName}
										class="user-avatar"
									/>
									<div class="user-info">
										<div class="user-name">{violation.user.firstName}</div>
										<div class="user-username">@{violation.user.username}</div>
									</div>
									<div class="action-badge {violation.actionTaken}">
										{#if violation.actionTaken === 'banned'}
											<XCircle size={14} />
										{:else}
											<AlertTriangle size={14} />
										{/if}
										<span>{violation.actionTaken}</span>
									</div>
								</div>

								<div class="violation-message">{violation.message}</div>

								<div class="violation-footer">
									<div class="violation-type">{violation.violationType}</div>
									<div class="violation-time">
										<Clock size={14} />
										<span>{formatDate(violation.createdAt)}</span>
									</div>
								</div>
							</div>
						{/each}
					</div>

					<!-- Violations Pagination -->
					{#if totalViolationsPages > 1}
						<div class="pagination">
							<button
								class="pagination-btn"
								disabled={violationsPage === 1}
								onclick={() => (violationsPage = Math.max(1, violationsPage - 1))}
							>
								<ChevronLeft size={18} />
								<span>Previous</span>
							</button>
							<div class="pagination-info">
								<span
									>Page {violationsPage} of {totalViolationsPages}</span
								>
							</div>
							<button
								class="pagination-btn"
								disabled={violationsPage === totalViolationsPages}
								onclick={() =>
									(violationsPage = Math.min(totalViolationsPages, violationsPage + 1))}
							>
								<span>Next</span>
								<ChevronRight size={18} />
							</button>
						</div>
					{/if}
				</div>

				<!-- Top Users -->
				<div class="users-panel">
					<div class="panel-header">
						<div class="panel-title">
							<Users size={20} />
							<h3>Top Violators</h3>
						</div>
						<div class="panel-badge">By warnings</div>
					</div>

					<div class="users-list">
						{#each paginatedUsers as user (user.id)}
							<div class="user-card">
								<img
									src={getAvatarUrl(user.username)}
									alt={user.firstName}
									class="user-avatar-large"
								/>
								<div class="user-details">
									<div class="user-name-large">{user.firstName}</div>
									<div class="user-username-small">@{user.username}</div>
								</div>
								<div class="user-warnings">
									<div class="warnings-count">{user.warningCount}</div>
									<div class="warnings-label">warnings</div>
								</div>
							</div>
						{/each}
					</div>

					<!-- Users Pagination -->
					{#if totalUsersPages > 1}
						<div class="pagination">
							<button
								class="pagination-btn"
								disabled={usersPage === 1}
								onclick={() => (usersPage = Math.max(1, usersPage - 1))}
							>
								<ChevronLeft size={18} />
								<span>Previous</span>
							</button>
							<div class="pagination-info">
								<span>Page {usersPage} of {totalUsersPages}</span>
							</div>
							<button
								class="pagination-btn"
								disabled={usersPage === totalUsersPages}
								onclick={() => (usersPage = Math.min(totalUsersPages, usersPage + 1))}
							>
								<span>Next</span>
								<ChevronRight size={18} />
							</button>
						</div>
					{/if}
				</div>
			</div>
		</div>
	</section>

	<!-- Footer -->
	<footer class="footer">
		<div class="container">
			<div class="footer-content">
				<div class="footer-text">
					<p>Powered by Group Guard API - Real-time Telegram Group Moderation</p>
					<p class="footer-subtext">Keeping communities safe with AI-powered moderation</p>
				</div>
				<a href="https://kentom.co.ke" target="_blank" rel="noopener" class="footer-link">
					Created by Kentom
				</a>
			</div>
		</div>
	</footer>
{/if}

<style>
	/* Loading */
	.loading-container {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 1.5rem;
	}

	.loading-spinner {
		animation: spin 1s linear infinite;
		color: var(--color-primary);
	}

	@keyframes spin {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}

	/* Container */
	.container {
		max-width: 1280px;
		margin: 0 auto;
		padding: 0 2rem;
	}

	/* Hero */
	.hero {
		background: linear-gradient(135deg, #fef3c7 0%, #fcd34d 100%);
		padding: 5rem 0;
		text-align: center;
		position: relative;
	}

	.hero-content {
		max-width: 800px;
		margin: 0 auto;
	}

	.hero-badge {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.5rem 1rem;
		background: white;
		border-radius: 100px;
		font-size: 0.875rem;
		font-weight: 500;
		margin-bottom: 2rem;
		box-shadow: var(--shadow-sm);
	}

	.status-dot {
		width: 8px;
		height: 8px;
		border-radius: 50%;
		background: var(--color-text-light);
	}

	.status-dot.connected {
		background: var(--color-success);
		animation: pulse 2s infinite;
	}

	@keyframes pulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.5;
		}
	}

	.hero-title {
		font-size: 3.5rem;
		font-weight: 800;
		margin-bottom: 1.5rem;
		color: var(--color-text);
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 1rem;
	}

	.hero-icon {
		width: 64px;
		height: 64px;
		border-radius: 50px;
		object-fit: cover;
		box-shadow: var(--shadow-lg);
	}

	.hero-description {
		font-size: 1.25rem;
		color: var(--color-text-muted);
		margin-bottom: 2.5rem;
		line-height: 1.8;
	}

	.hero-features {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem;
		justify-content: center;
		margin-bottom: 2.5rem;
	}

	.feature-item {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.75rem 1.25rem;
		background: white;
		border-radius: var(--radius);
		font-weight: 500;
		color: var(--color-text);
		box-shadow: var(--shadow-sm);
	}

	.feature-item :global(svg) {
		color: var(--color-success);
	}

	.kentom-link {
		display: inline-flex;
		align-items: center;
		gap: 0.75rem;
		padding: 0.75rem 1.5rem;
		background: var(--color-text);
		color: white;
		border-radius: var(--radius);
		text-decoration: none;
		font-weight: 600;
		transition: transform 0.2s;
		box-shadow: var(--shadow);
		position: absolute;
		bottom: 1rem;
		left: 1rem;
	}

	.kentom-link:hover {
		transform: translateY(-2px);
		box-shadow: var(--shadow-lg);
	}

	.kentom-logo {
		width: 24px;
		height: 24px;
	}

	/* Stats Section */
	.stats-section {
		padding: 4rem 0;
		background: var(--color-bg-alt);
	}

	.section-title {
		font-size: 2rem;
		font-weight: 700;
		text-align: center;
		margin-bottom: 3rem;
		color: var(--color-text);
	}

	.stats-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
		gap: 1.5rem;
	}

	.stat-card {
		background: var(--color-surface);
		padding: 2rem;
		border-radius: var(--radius-lg);
		display: flex;
		align-items: center;
		gap: 1.5rem;
		border: 1px solid var(--color-border);
		transition: all 0.3s;
	}

	.stat-card:hover {
		transform: translateY(-4px);
		box-shadow: var(--shadow-lg);
		border-color: var(--color-primary);
	}

	.stat-icon {
		width: 56px;
		height: 56px;
		border-radius: var(--radius);
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}

	.stat-icon.users {
		background: #dbeafe;
		color: #1e40af;
	}

	.stat-icon.warnings {
		background: #fef3c7;
		color: #d97706;
	}

	.stat-icon.violations {
		background: #fce7f3;
		color: #be185d;
	}

	.stat-icon.bans {
		background: #fee2e2;
		color: #dc2626;
	}

	.stat-content {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.stat-label {
		font-size: 0.875rem;
		color: var(--color-text-muted);
		font-weight: 500;
	}

	.stat-value {
		font-size: 2rem;
		font-weight: 700;
		color: var(--color-text);
	}

	.stat-trend {
		color: var(--color-success);
	}

	/* Activity Banner */
	.activity-banner {
		background: linear-gradient(90deg, #fef3c7 0%, #fde68a 100%);
		padding: 1rem 0;
		border-top: 2px solid var(--color-primary);
		border-bottom: 2px solid var(--color-primary);
		animation: slideDown 0.3s ease-out;
	}

	@keyframes slideDown {
		from {
			opacity: 0;
			transform: translateY(-20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.activity-content {
		display: flex;
		align-items: center;
		gap: 1rem;
		font-size: 0.95rem;
	}

	.activity-icon {
		color: var(--color-primary-dark);
		animation: pulse 2s infinite;
	}

	.activity-info {
		flex: 1;
		color: var(--color-text);
	}

	.activity-time {
		color: var(--color-text-muted);
		font-size: 0.875rem;
	}

	/* Content Section */
	.content-section {
		padding: 4rem 0;
		background: var(--color-bg);
	}

	.content-grid {
		display: grid;
		grid-template-columns: 1fr 400px;
		gap: 2rem;
	}

	@media (max-width: 1024px) {
		.content-grid {
			grid-template-columns: 1fr;
		}
	}

	/* Panel */
	.panel-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 1.5rem;
	}

	.panel-title {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		color: var(--color-text);
	}

	.panel-title h3 {
		font-size: 1.5rem;
		font-weight: 700;
	}

	.panel-badge {
		padding: 0.5rem 1rem;
		background: var(--color-bg);
		border-radius: var(--radius);
		font-size: 0.875rem;
		font-weight: 500;
		color: var(--color-text-muted);
		border: 1px solid var(--color-border);
	}

	/* Violations */
	.violations-list {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.violation-card {
		background: var(--color-surface);
		padding: 1.5rem;
		border-radius: var(--radius-lg);
		border: 1px solid var(--color-border);
		transition: all 0.2s;
		animation: slideUp 0.3s ease-out;
	}

	@keyframes slideUp {
		from {
			opacity: 0;
			transform: translateY(20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.violation-card:hover {
		border-color: var(--color-primary);
		box-shadow: var(--shadow);
	}

	.violation-header {
		display: flex;
		align-items: center;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.user-avatar {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		border: 2px solid var(--color-border);
	}

	.user-info {
		flex: 1;
	}

	.user-name {
		font-weight: 600;
		color: var(--color-text);
	}

	.user-username {
		font-size: 0.875rem;
		color: var(--color-text-muted);
	}

	.action-badge {
		display: flex;
		align-items: center;
		gap: 0.375rem;
		padding: 0.375rem 0.75rem;
		border-radius: 100px;
		font-size: 0.75rem;
		font-weight: 600;
		text-transform: uppercase;
	}

	.action-badge.warned {
		background: #fef3c7;
		color: #d97706;
	}

	.action-badge.banned {
		background: #fee2e2;
		color: #dc2626;
	}

	.violation-message {
		padding: 1rem;
		background: var(--color-bg);
		border-radius: var(--radius);
		font-size: 0.95rem;
		color: var(--color-text);
		margin-bottom: 1rem;
		line-height: 1.6;
	}

	.violation-footer {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
	}

	.violation-type {
		padding: 0.25rem 0.75rem;
		background: #fef3c7;
		color: var(--color-primary-dark);
		border-radius: 100px;
		font-size: 0.75rem;
		font-weight: 600;
		text-transform: capitalize;
	}

	.violation-time {
		display: flex;
		align-items: center;
		gap: 0.375rem;
		font-size: 0.875rem;
		color: var(--color-text-muted);
	}

	/* Users */
	.users-list {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.user-card {
		background: var(--color-surface);
		padding: 1.25rem;
		border-radius: var(--radius-lg);
		border: 1px solid var(--color-border);
		display: flex;
		align-items: center;
		gap: 1rem;
		transition: all 0.2s;
		animation: slideUp 0.3s ease-out;
	}

	.user-card:hover {
		border-color: var(--color-primary);
		box-shadow: var(--shadow);
	}

	.user-rank {
		width: 36px;
		height: 36px;
		border-radius: 50%;
		background: var(--color-primary);
		color: white;
		display: flex;
		align-items: center;
		justify-content: center;
		font-weight: 700;
		font-size: 0.875rem;
	}

	.user-avatar-large {
		width: 48px;
		height: 48px;
		border-radius: 50%;
		border: 2px solid var(--color-border);

	/*	ring of shadow ring-2*/
		box-shadow: 0 0 0 2px var(--color-primary);
	}

	.user-details {
		flex: 1;
	}

	.user-name-large {
		font-weight: 600;
		color: var(--color-text);
		margin-bottom: 0.125rem;
	}

	.user-username-small {
		font-size: 0.875rem;
		color: var(--color-text-muted);
	}

	.user-warnings {
		text-align: right;
	}

	.warnings-count {
		font-size: 1.75rem;
		font-weight: 700;
		color: var(--color-primary);
	}

	.warnings-label {
		font-size: 0.75rem;
		color: var(--color-text-muted);
		text-transform: uppercase;
		letter-spacing: 0.5px;
	}

	/* Footer */
	.footer {
		background: var(--color-bg-alt);
		padding: 3rem 0;
		border-top: 1px solid var(--color-border);
		margin-top: 4rem;
	}

	.footer-content {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 2rem;
	}

	@media (max-width: 768px) {
		.footer-content {
			flex-direction: column;
			text-align: center;
		}
	}

	.footer-text p {
		color: var(--color-text-muted);
		margin-bottom: 0.5rem;
	}

	.footer-subtext {
		font-size: 0.875rem;
		color: var(--color-text-light);
	}

	.footer-link {
		color: var(--color-primary);
		text-decoration: none;
		font-weight: 600;
		transition: color 0.2s;
	}

	.footer-link:hover {
		color: var(--color-primary-dark);
	}

	/* Responsive */
	@media (max-width: 768px) {
		.hero {
			padding: 3rem 0;
		}

		.hero-title {
			font-size: 2.5rem;
		}

		.hero-icon {
			width: 48px;
			height: 48px;
		}

		.hero-description {
			font-size: 1.1rem;
		}

		.section-title {
			font-size: 1.75rem;
		}

		.stat-value {
			font-size: 1.5rem;
		}

		.content-grid {
			gap: 3rem;
		}
	}
</style>
