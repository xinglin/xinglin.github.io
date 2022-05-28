# Dynamic Kernel Debugging


0. Compile kernel with CONFIG_DYNAMIC_DEBUG=y  
1. Mount debugfs, if not mounted yet  

		sudo mount -t debugfs none /sys/kernel/debug
	
2. Turn on debug for a specific module  

		echo 'module pblk +pfl' > /sys/kernel/debug/dynamic_debug/control

The flags are:

		p    enables the pr_debug() callsite.
		f    Include the function name in the printed message
		l    Include line number in the printed message
		m    Include module name in the printed message
		t    Include thread ID in messages not generated from interrupt context
		_    No flags are set. (Or'd with others on input)

